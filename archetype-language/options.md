# Composite types

## Options

An option is convenient when a variable may or may not have a value of a given type.

#### Declare&#x20;

`some` and`none` are the 2 operators to build an option value.

```css
variable receiver : option<address> = none // to be set later ...

variable message : option<string> = some("there is a message")
```

#### Test&#x20;

`issome` and `isnone` are the 2 operators to test whether an option value is none or is some.

```css
entry testopt (a : option<int>) {
  requires {
     r1 : issome(a);
  }
  failif {
     f1 : isnone(a);
  }
  effect {
    ...
  }
}
```

#### Get&#x20;

`opt_get` is the operator to extract the value from a `some` option value. It _fails_ if the value is `none`.

```css
effect {
  var a = some("a string");
  if issome(a) then (
    if opt_get(a) = "a string" then transfer 1tz to coder;
  );
}
```

## Tuples

Tuples are like _vectors_ of values. For example, the following declares a tuple of three values, an integer on the first dimension, a string on the second and a bytes on the third:

```css
variable t : int * string * bytes = (2020, "is the year of", 0x050100000009617263686574797065)
```

The tuple type is built with `*` and the tuple value with `(,)` syntax.

The `[ ]` operator is used to project the tuple to one specific dimension:

```css
effect {
   var t = (2020, "is the year of", 0x050100000009617263686574797065);
   var y = t[0]; /* 2020 */
   var s = t[1]; /* "is the year of" */
   var b = t[2]; /* 0x050100000009617263686574797065 */
}
```

Projection _fails_ if the dimension index is invalid.&#x20;

The dimension index _must_ be a straightforward literal of integer. It does not support any expression, even an expression that would reduce to an integer value. For example, the following does not compile:

```css
var y = t[1-1]; /* does NOT compile! */
```

## Records

A record is a list of named values. For example, the following declares a record of two values:

```css
record geoloc {
  longitude : rational;
  latitude : rational;
}
```

It is possible to read and assign values to the fields of a record:

```css
var r : geoloc = { /* Eiffel tower's location */
  longitude = 48.8583701; 
  latitude = 2.2944813 
}; 
var lo = r.longitude; /* lo = 48.8583701 */
var la = r.latitude; /* la = 2.2944813 */
/* let's go to another Eiffel's construction! */
r.longitude := 40.6892494;
r.latitude := 74.04;
```

### Enums

An enum is a union type, that is a value taken from a list of possible values. For example the the following enum defines 3 possible colors:

```ocaml
enum color = 
| Red
| Green
| Blue
```

You can then create a value by using one of the three possible values:

```ocaml
var c = Red;
```

You can test the value with \`\`
