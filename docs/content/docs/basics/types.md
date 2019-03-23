---
weight: 3
---

# Types

Type is an overloaded term in _lang_.

There are 3 forms of types: the "type actual" which refer to built-in types,
the `type` function (see <a>Custom Type </a>), and the "interface" type (see
<a>Interface</a>)

## Defining types

Syntax is

```
binding_name : <type> ...
```

## Runtime type binding

Anything can be a type

## Built-In

This page defines the built-in type

### primitives

```
int   => 
float => float
bool  => boolean
list  => list
```

There's also collection types

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

#### Index access

You can access an item in a list via an index as you would suspect

```
collection = [1 , 2 , 3]

collection[0]
=> 1

collection[2]
=> 3
```

You can also select from the back of a list using negative indices

```
collection = [1 , 2 , 3]

collection[-1]
=> 3

collection[-2]
=> 2
```

#### Slice and dice

You can also slice up a lists using the `:`

Get everything but the first item
```
collection = [1 , 2 , 3, 4, 5, 6]

slice = collection[1:]
// => [2, 3, 4, 5, 6]
```

Get from `start` to `stop`

```
slice = collection[2:4]
// => [3, 4, 5]
```

Slices are inclusive by default but you can change that via a config

```
config.exclusive_slice {
  collection = [1 , 2 , 3, 4, 5, 6]

  slice = collection[2:4]
  // => [4]
}
```

#### Ranges

You can also define lists via a ranage syntax

```
collection = [1..99]
// => [1, 2, 3 .. 99]
```

#### Functions

_lang_ ships with some functions that make accessing list memebers more
declarative

Grab the first item of a list

```
collection = [1, 2, 3, 4]
x = first collection
// => 1

// Or
x = collection.first
// => 1
```

Grab the last item of a list

```
collection = [1, 2, 3, 4]
x = last collection
// => 1

// Or
x = collection.last
// => 1
```

You can get the _head_ and _tail_ of a list easily via destructuring

```
head, tail... = [1, 2, 3, 4]
// => head 1
// => tail [2, 3, 4]
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

#### Shorthand

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
