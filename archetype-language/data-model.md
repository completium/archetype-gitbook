---
description: Variable and asset collection
---

# Assets

## Constant

A constant is a global value that cannot be changed. The following snippet defines a global constant named `price` typed `tez` equal to 10 tez:

```ocaml
constant price : tez = 10tz
```

The following defines a `string` typed constant named `msg` :

```ocaml
constant msg : string = "Hello world"
```

## Variable

A variable is a global value that can be changed. The following snippet defines 2 global variables named `seller` and `buyer` typed role equal to a (here random) addresses:

```ocaml
variable seller : role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk
variable buyer  : role = @tz1KUbontHGor25CztkksC8RvjUWAbXYJuNr
```

## Asset

An archetype contract can store asset data. An asset is described with a set of fields, one of which is the asset identification field. Each asset has a unique identification value.

For example, the following defines a car asset:

```ocaml
asset car identified by vin {
  vin : string;
  model : string;
  year : int;
}
```

By default, assets are sorted on the identifier value. It is possible to specify another default sort. For example to sort car assets on the `year` field value:

```
asset car identified by vin sorted by year {
  vin : string;
  model : string;
  year : int;
}
```

The type of a field may be a basic type, or one of `collection` or `partition`.

A collection is the generic name for an asset container. For example, if `part` is the asset for vehicle part, it is possible to describe which parts a car is made of:

```ocaml
asset part identified by pid {
   pid : string;
}

asset car identified by vin {
  vin : string;
  model : string;
  year : int;
  parts : aggregate<part>;
}
```

Say now that the contract deals with animals and country fields. An animal is in one, and only one field (obviously). A fields has a collection of animals. But rather than `collection`, the `partition` keyword specifies that _every_ animal is in _one_ field:

```ocaml
asset animal identified by aid {
  aid : string
}

asset field identified by fid {
  fid : string;
  animals : partition<animal>
}
```

Declaring an asset implicitly creates the collection which contains asset instances. It means that data storage in Archetype is abstract, like SQL, compared to programming languages which provides set, map, list, ... containers.

In order to get a technical vision of asset collections, you may consider that the asset collection is equivalent to a map between the identifier field and a record made of the other fields.

For example, the Ocaml equivalent of the `car` asset declaration is a record declaration plus a map declaration.

```ocaml
type car = {
  model : string;
  year : int;
}

module CarCollection = Map.make(String)
```

Storage of asset is abstract in Archetype in order for the transcoder to apply optimisation strategy when generating execution code. For example, if the car collection is only accessed via get and set methods, then it will be generated as a _set_ container.
