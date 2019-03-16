---
weight: 3
---

# Built-in Types

### Lists

A list can hold stuff.

You can have the same stuff

```
collection = [1, 2, 3]
```

You can have different stuff

```
collection = [1, "foobar", 3, false]
```

You can be explicit about stuff

```
collection:list:int = [1, 2, "boo"]

// => Compile error: The list 'collection' can only have 'int' ....
```

### Structures

The structures in _lang_ are basically JSON

```
data = {
  foo: 'bar',
  num: 123,
}
```

This should create a structure called `data` with fields `foo` of type string and
`bar` of type integer

You should not be limited to primitive types, you can also add functions as fields

```
data = {
  foo: 'bar',
  num: 123,
  f: a b -> a + b,
}

```

Here `data` now also has a function associated to it, `f`

You can call a function when declaring a structure

```
data = {
  foo: 'bar',
  num: 123,
  pi: (-> 3.14)()
}
```

This will add `pi` with the float value of `3.14` to `data` structure

#### Shorthands

You can pass in variables and the structure will be assigned key:value pairs
using the variable name and value

```
foo = 'bar'
pi -> 3.14

data = {
  num: 456,
  foo,
  pi
}

=> data {num: 456, foo: 'bar', pi: () -> Int}
```

`pi` in the above example is a function that returns pi. If you want the
actual value of the function you'll have to call the function

```

data = {
  num: 456,
  foo,
  pi()
}

=> data {num: 456, foo: 'bar', pi: 3.14}
```
