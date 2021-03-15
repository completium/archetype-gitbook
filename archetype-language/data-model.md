---
description: Variable and asset collection
---

# Assets

## Declaration

An asset is defined by a set of fields, one of which is the identification field. Each asset has a unique identification value over the collection of assets.

For example, the following defines a car asset:

```coffeescript
asset car identified {
  vin     : string;
  model   : string;
  year    : int;
  nbdoors : int;
}
```

By default, the first field is used as the asset _identifier_. It is possible to specify another field with the `identified by` instruction. It is also possible to declare several fields to identify an asset:

```coffeescript
asset allowance identified by owner spender {
  owner       : address;
  spender     : address;
  amount      : nat;
}
```

## Collection

Declaring an asset automatically creates a collection of these assets. This collection is named after the asset. In the example above,  `car` refers to the collections of car assets. In a collection, each asset has a unique identifier.

### Write

#### add

Use the `add` instruction to add an asset to the collection. It _fails_ if an asset with the same id is already in the collection:

```ocaml
effect {
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
}
```

#### update

Use the `update` instruction to update a field of an asset. It _fails_ if the collection does not have an asset with the id:

```javascript
effect {
   /* update one field */
   car.update("YS3ED48E5Y3070016", { year = 1967 });
   /* update severa fields */ 
   car.update("YS3ED48E5Y3070016", { year = 1967; nbdoors = 3 });
   /* update all fields, no need for field name */
   car.update("YS3ED48E5Y3070016", { "fiesta"; 2020; 4 });
}
```

It is possible to increment or decrement integer fields with the `+=` and `-=` operators:

```ocaml
effect {
  car.update("YS3ED48E5Y3070016", { year += 2; nbdoors -= 1 });
}
```

#### addupdate

Use the `addupdate` instruction to add or update an update. The argument asset is added if not already present or updated if present. It does _not_ fail.

```javascript
effect {
   // all fiels are necessary, no field name required
   car.addupdate("YS3ED48E5Y3070016", {  model = "mustang"; 
                                         year = 1968; 
                                         nbdoors = 2 });
}
```

#### remove

Use the `remove` instruction to remove an asset. 

```ocaml
effect {
   car.remove("YS3ED48E5Y3070016");
}
```

{% hint style="info" %}
`remove`does _**not**_ fail if the collection does not have an asset with the id.
{% endhint %}

#### removeif

Use the `removeif` instruction to remove assets under given criteria. For example, in order to remove all cars with less than 4 doors:

```ocaml
effect {
  car.removeif(the.nbdoors < 4);
}
```

The `the` keyword refers to the asset being evaluated. 

### Read an asset

The `[ ]` and `.` syntax is used to retrieve an asset and read a field value: 

```ocaml
effect {
  var amodel = car["YS3ED48E5Y3070016"].model;
  ...
}
```

## Views

Views are used to access assets in a different way than the base collection seen above. They enable access with a different order, position range, or specific criteria.

You cannot build a view from scratch. It is derived from a collection with the operators presented below. Note at last that you cannot modify the collection through a view.

As for collections, views guarantee the order of the assets. By default, assets are ordered by identification field values, unless an explicit `sort` is invoked.

Note that collections are automatically transformed to views when necessary. As a consequence, you may use the view API presented below straightforwardly on collections; for example `car.count()` returns the number of car asset in the car collection.

### Create

#### head

`head` returns the first n elements of the collection or view. By default assets are ordered by the identification field. It _fails_ if n is greater than the number of elements in the collection or view \(or negative\).

```javascript
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   car.add({ "1FTDF15Y2SNB02216"; "focus"; 2008; 4});
   var v = car.head(2); // view creation :
                        // v has 1FTDF15Y2SNB02216 and 3VWCK21Y33M306146 assets
   ...
}
```

#### tail

`tail` returns the _last_ n elements of the collection or view. By default assets are ordered by the identification field. It _fails_ if n is greater than the number of elements in the collection or view \(or negative\).

```javascript
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   car.add({ "1FTDF15Y2SNB02216"; "focus"; 2008; 4});
   var v = car.tail(1); // view creation :
                        // v has only the YS3ED48E5Y3070016 asset
   ...
}
```

#### sort

`sort` provides access to assets in a different order than the default one based on the identification order. 

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   car.add({ "1FTDF15Y2SNB02216"; "focus"; 2008; 4});
   var v = car.sort(nbdoors);
   if v.nth(0) = "YS3ED48E5Y3070016" then transfer 1tz to coder;
   if v.nth(1) = "1FTDF15Y2SNB02216" then transfer 1tz to coder;
   if v.nth(2) = "3VWCK21Y33M306146" then transfer 1tz to coder;
}
```

Note in the example below that the sort operator preserves the initial order for assets with the same criterion. In this example, `1FTDF15Y2SNB02216` comes before `3VWCK21Y33M306146`.

It is possible to specify the descending order with the `desc` operator:

```ocaml
var v = car.sort(desc(nbdoors));
```

`sort` is _**not**_ stable; it means that two assets with equal sort values do not appear in the same order in sorted output as they appear in the input view.

As a consequence, if you want to sort a collection according to two fields `f1`and `f2`, the following will not provide what you want:

```javascript
myasset.sort(f1).sort(f2);
```

The solution is to pass the fields as arguments to the `sort` operator:

```javascript
myasset.sort(f1,f2);
```

{% hint style="info" %}
Do _not_ chain `sort` operations; rather call one `sort` operation with the fields as arguments.
{% endhint %}

#### select

`select` returns a view with assets that satisfy a criterion:

```javascript
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   car.add({ "1FTDF15Y2SNB02216"; "focus"; 2008; 4});
   var v = car.select(the.nbdoors = 4); // view creation
                                        // v contains 3VWCK21Y33M306146 and 
                                        // 3VWCK21Y33M306146 assets
   ...
}
```

As in `removeif` above, the `the` keyword refers to the asset being evaluated. 

### Test membership

`contains` returns true if the view or collection contains a key. 

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   if car.contains("YS3ED48E5Y3070016") then transfer 1tz to coder;
}
```

### Access by position

`nth` returns the key at position n in the view. By default assets are ordered by the identification field. It _fails_ if n is out of the bounds of the collection.

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   if car.nth(0) = "3VWCK21Y33M306146" then transfer 1tz to coder; 
   if car[car.nth(0)].model = "mustang" then transfer 1tz to coder;
}
```

### Summerize

#### count

`count` returns the number of elements in a collection:

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   if car.count() = 1 then transfer 1tz to coder;
}
```

#### sum

`sum` returns the sum of a field:

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   car.add({ "1FTDF15Y2SNB02216"; "focus"; 2008; 4});
   if car.sum(nbdoors) = 10 then transfer 1tz to coder;
}
```

### Iterate

It is possible to iterate a view with the `for ... in ... do ... end` loop instruction.

```ocaml
effect {
   for k in car.select(the.year >= 2010) do
     car.update(k, { nbdoors += 1 } );
   end;
}
```

The example above increments `nbdoors` of cars with `year` above 2010.

Note that the iteration is done over asset keys.

### Clear

Use the `clear`  instruction to remove all asset referenced in a view:

```javascript
effect {
   car.select(the.year = 2000).clear();
}
```

{% hint style="info" %}
`clear` is the only view instruction that changes the contract data.
{% endhint %}

