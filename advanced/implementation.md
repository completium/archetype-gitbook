# Implementation

Archetype is an open source project developed in [Ocaml](https://ocaml.org/index.html).

{% embed url="https://github.com/edukera/archetype-lang" caption="github repository" %}

## Transcoding process

The archetype compiler/transcoder reads an archetype source file \(with "arl" extension\) and generates transcoded versions of the contract in different languages.

The first version provides [PascalLIGO](http://ligolang.org/) syntax for execution, and [Why3](http://why3.lri.fr/) syntax for verification.

This section describes the different steps of the transcoding process.

### Lexing / Parsing

The parse tree is generated with [menhir](http://gallium.inria.fr/~fpottier/menhir/) generated parsing module. It is done with error recovery mechanism in order to be compliant with [LSP](https://microsoft.github.io/language-server-protocol/) \(for integration in IDE\).

Thank you to Yann Regis Gianas for setting up a example of error recovery mechanism with menhir:

{% embed url="https://github.com/yurug/menhir-error-recovery" caption="" %}

### Typing

The parse tree is then checked for basic programming errors \(e.g. unknown variables, non well-formed expressions, etc...\) and assigns types \(i.e. structural informations\) to the different parts of the contracts. This allows to check that the contract satisfies some well-formedness rules and reduces the possibility of bugs in the contract.

This phase outputs a _typed AST_ \(Abstract Syntax Tree\) that is at the root of the next phases.

Thank you to Pierre Yves Strub for the implementation.

### Intermediate language

The typed AST is mapped to an intermediate language \(IL\) with an explicit storage structure.

All constants, variables and assets are held in that storage structure and contract's data actions \(get, set, remove, ...\) is mapped to a predefined storage API.

Transitions and special actions' instruction \(called by, accept transfer, ...\) are reduced to basic `if ... then fail ... else ...` instructions.

For example, consider the following transition from the escrow contract:

```ocaml
transition complete () from Confirmed {
  called by oracle

  to Completed when { now < deadline }
  with effect {
    transfer price to seller
  }
}
```

The IL version is presented below:

```ocaml
let complete = 
  match s.state with
  | Confirmed ->
      if caller <> oracle then fail ("invalid caller");
      if now < s.deadline then
       transfer s.price to seller
      else fail ("invalid condition")
  | _ -> fail ("invalid state")
```

`s` is the static storage variable.

IL is the language to start from when transcoding to a new language. The [Printer\_model](https://github.com/edukera/archetype-lang/blob/master/src/printer_model.ml) module pretty prints the IL representation.

Printing the IL of a contract is the default action of the archetype compiler; the following prints the IL for the contract [_basic\_escrow.arl_](../contract-library/escrow/basic-escrow.md):

```text
$ archetype basic_escrow.arl
```

### IL transforms

Target languages \(PascalLIGO, SmartPy, why3 ml, ...\) are obtained by applying transforms to the IL. This section presents the main transforms.

#### Side effect removal

Archetype language has side effects. So does IL. When the target language does not support side effect, like in pure functional programming languages, side effect has to be removed.

For example, in the above complete function, the transfer function makes a side effect. The version without side effect is a follows:

```ocaml
let complete = 
  let t = 
    match s.state with
    | Confirmed ->
        if caller <> oracle then fail;
        if now < s.deadline then
          let t = transfer s.price seller in
          t
        else fail
  | _ -> fail in
  t
```

#### Split key-value

In order to minimise the necessary computation to access an asset from its key, It is necessary to store assets in a map which associates the key to the asset record.

With this transform, assets are stored in maps, as exemplified below with the generated storage of the [miles with expiration](../contract-library/tokens/miles-with-expiration.md) contracts :

```ocaml
record mile identified by id {
  id : string;
  amount : int;
  expiration : date;
}

record owner identified by addr {
  addr : role;
  miles : mile partition;
}

storage {
  admin : address = tz1aazS5ms5cbGkb6FN1wvWmN7yrMTTcr6wB;
  mile_keys : string collection = [];
  mile_assets : (string, mile) map = [];
  owner_keys : role collection = [];
  owner_assets : (role, owner) map = [];
}
```

The other impact of this transform is to consider that all operations on asset collections \(select, sum, max, ...\) take a collection of keys for argument, rather than a collection of assets.

### Printers

A printer pretty-prints the IL as the target language.

#### PascaLIGO

The IL with the no-side-effect and asset-shallowing transforms is very close to the PascaLIGO format. It just needs to be pretty printed.

#### Why ml

Why ml is the why3 format. Like ocaml, it accepts side effects. Hence no need for the no-side-effect transform.

On the logical side, it is necessary to compute asset invariants' preconditions for the storage API.

Indeed, an asset invariant is transformed to a storage invariant. As such, one of the verification tasks generated by the why3 framework, is the verification that the invariant holds as the postcondition of any function, given it holds as a precondition.

When adding a new asset to an asset collection, the new asset must comply to the invariant in some way; more specifically, in the way that, when added, the invariant stills holds. The precondition on the new asset sufficient for the invariant to hold when added, must be figured out.

A way to compute this precondition is the [Weakest Precondition](https://en.wikipedia.org/wiki/Predicate_transformer_semantics#Weakest_preconditions) \(WP\) Calculus. Hence, for every storage API entry, the precondition has to be generated with WP calculus, for the verification task to succeed.

The Why3 framework, while based on WP, does not provide a way to generate these preconditions, nor inlining function calls \(which would be equivalent\). Hence the Archetype transcoder implements the WP calculus to generate, for each entry of the storage API, the precondition corresponding to an asset invariant.

For example, consider the following asset with the invariant property on the `amount` value:

```ocaml
asset mile identified by id {
   id         : string;
   amount     : int;
   expiration : date;
} with {
  i : amount > 0;
}
```

The `add_mile` API entry in why3 ml is generated as follows:

```python
let add_mile (s : storage) (new_mile : mile) : unit =
    if mem (id new_mile) s.mile_keys then
        raise KeyExist
    else
        s.mile_assets <- set s.mile_assets (id new_mile) new_mile;
        s.mile_keys <- add (id new_mile) s.mile_keys
```

The WP calculus for the invariant `amount > 0` through the above function, gives the following property:

```python
not (mem (id new_mile) s.mile_keys) ->
forall k : key. 
k = id new_mile \/ mem k (s.mile_keys) -> 
amount (get (set s.mile_assets (id new_mile) new_mile) k) > 0
```

This property is added as a precondition of the `add_mile` function:

```python
let add_mile (s : storage) (new_mile : mile) : unit
    requires { 
        not (mem (id new_mile) s.mile_keys) ->
        forall k : key. 
        k = id new_mile \/ mem k (s.mile_keys) -> 
        amount (get (set s.mile_assets (id new_mile) new_mile) k) > 0
    }
    =
    if mem (id new_mile) s.mile_keys then
        raise KeyExist
    else
        s.mile_assets <- set s.mile_assets (id new_mile) new_mile;
        s.mile_keys <- add (id new_mile) s.mile_keys
```

So equipped, the add\_mile function will propagate the precondition on the added mile whenever the function is called.

