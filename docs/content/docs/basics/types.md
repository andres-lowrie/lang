---
weight: 3
---

# Types

Type is an overloaded term in _lang_.

There are 3 forms of types: 

- The built-in types
- The `type` function (see <a>Custom Type </a>)
- The "interface" type (see <a>Interface</a>)

## Defining types

Syntax is

```
binding_name [ : <type> ... ] = assignment
fn binding_name [ : <type> ... ] -> <type> block
```

For example

```
// In simple assignment
foo:int = 0

// When declaring function parameters
foo a:int b:int -> int {}
```

Some types are "Complex types" eg structures, lists, and potentially custom
types in that they contain other data. For these types, you specify containing
type by simply adding more `:<type>`

```
// A list that holds strings
foo:list:str = [...]

// A list ... that holds lists.... that holds strings
foo:list:list:str = [[]...]
```

The "type chaining" syntax works as long as the thing on the left of the `:`
can accept a type so for example the following wouldn't work

```
foo:str:list = ...
=> Compile Error: The type `str` doesn't accept types....
```

### Anything can be a type

...Including ...yup... functions

You can pass:

Any function that `type` spits out (see <a>Custom Types</a>)

Any function that `interface` spits out (see <a>Interfaces</a>)

Any function that takes a single parameter that returns a boolean ie: a
function with the following signature

```
a:anything -> bool
```

For example:

```
// Here foo is a value function in that it will always return true regardless
of what is passed into it
foo -> true

// Meaning that when "foo" is used as a type, _lang_ will let anything be
assigned to the variable on the left hand side

x:foo = 1
x:foo = 'x'
x:foo = []
```

As long as the function has this signature (or), it doesn't matter what the function
does _lang_ will run the function passing whatever is on the right hand side of
the "=" and check the return value; if it's `true` lang will allow the
assignment

For example:

```
even a -> mod a 2 is 0
odd a -> not even

// Now we can say that "x" should only ever be a value that's even
// and "y" should be odd

x:even = {...}
y:odd = {...}
```

And you can also pass any function regardless of arity, however _lang_ will go
down a different decision path when it encounters functions that take more than
1 parameter

Note 


Eg:

```
// "a" is defined to take no parameters
a -> 1

foo:a
```




You can also negate types, as in "this variable can be anything except a `type`"

```
foo:!int
```

You can also set a strict type... dynamically


## Built-In

These are the types that are built into the language, while you can "extend"
these you cannot delete these

### Primitives 

The classics

```
int      => integer
float    => float
bool     => boolean
str      => string
anything => Anything
nothing  => Nothing
```

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

=> Compile error: The list 'collection' can only have 'int' ....
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
=> [2, 3, 4, 5, 6]
```

Get from `start` to `stop`

```
slice = collection[2:4]
=> [3, 4, 5]
```

Slices are inclusive by default but you can change that via a config

```
config.exclusive_slice {
  collection = [1 , 2 , 3, 4, 5, 6]

  slice = collection[2:4]
  => [4]
}
```

#### Ranges

You can also define lists via a ranage syntax

```
collection = [1..99]
=> [1, 2, 3 .. 99]
```

#### Functions

_lang_ ships with some functions that make accessing list memebers more
declarative

Grab the first item of a list

```
collection = [1, 2, 3, 4]
x = first collection
=> 1

// Or
x = collection.first
=> 1
```

Grab the last item of a list

```
collection = [1, 2, 3, 4]
x = last collection
=> 1

// Or
x = collection.last
=> 1
```

You can get the _head_ and _tail_ of a list easily via destructuring

```
head, tail... = [1, 2, 3, 4]
=> head 1
=> tail [2, 3, 4]
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
