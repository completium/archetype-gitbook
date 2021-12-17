# Transfers, contract calls

## Transfer

The `transfer` instruction is used to transfer Tezis to an account.

```javascript
effect {
  transfer 1tz to coder;
}
```

where `coder` is a `role` variable.

## Calling a contract

It is possible to call a smart contract with the `transfer` instruction. Say for example you want to call the `add_value` entry point of the following contract: 

```javascript
archetype contract_called

variable v : nat = 0

entry add_value(x : nat) { v += x }
```

The contract may then be called with the `transfer ... to ... call ...` instruction:

```javascript
effect {
    var c = @KT1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm;
    transfer 0tz to c call add_value<nat>(3);
}
```

Note that it _fails_ if the contract at address `c` does not provide an entry point annotated `%add_value`

### Self

It is possible to call the current contract itself. Say the current contract has the `my_add_value` entry point; it is possible to call it from another entry point with the following instruction:

```javascript
entry my_add_value(a : nat, b : nat) {
   sum := a + b;
}

entry add_1_3 () {
   transfer 0tz to entry self.my_add_value(1,3)
}
```

### Getter & contract

A common pattern for contracts to exchange values is to call an entry point with a callback.

This pattern is presented in the _two-way-inter-contract_ _invocation_ article below:

{% embed url="https://medium.com/protofire-blog/enabling-smart-contract-interaction-in-tezos-with-ligo-functions-and-cps-e3ea2aa49336" %}

The archetype version of the _sender_ contract \(see article above\):

```javascript
archetype sender

variable bar : nat = 0

getter getBar () { return bar }

entry setBar (b : nat) { bar := b }
```

It uses the `getter` keyword used to declare an entry point to the contract called "getBar"; the Michelson version of this entry actually takes a callback function \(a setter\) used to set/use the `bar` value in another contract. It is syntactic sugar equivalent to the following entry declaration:

```javascript
entry getBar (cb : contract<nat>) { transfer 0tz to entry cb(bar) }
```

The `contract` type is used to declare the callback type; it is parametrized by the signature of the callback, represented as the tuple of argument types.  

The difference between the `getter` and `entry` versions of the `getBar` entry above is that the callback argument is _anonymous_ in the `getter` version.

The archetype version of the _inspector_ contract \(see article above\):

```javascript
archetype inspector

variable foo : int = 0

entry setFoo(v : int) { foo := v }

entry getFoo(asender : address) { 
  transfer 0tz to asender call getBar<contract<int>>(self.setFoo) 
}
```

### Entrypoint

The `entrypoint` function may be used to build a `contract` value from the name and the address. It returns an _option_ value of contract type, so that it is possible to handle the case when the entry point does not exist.

The `require_entrypoint` function builds a contract and fails with error message if contract is invalid. The above _inspector_ contract may be rewritten as:

```javascript
archetype inspector

variable foo : int = 0

entry setFoo(v : int) { foo := v }

entry getFoo(asender : address) { 
  var e = require_entrypoint<int>("%getBar", asender, "invalid address");
  transfer 0tz to entry e(self.setFoo))
}
```



