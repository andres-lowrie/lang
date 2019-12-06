# Types

Type is an overloaded term in _lang_.

When the word "type" is used it is in reference to 5 things:

- The built-in types (what one would expect)
- The `type constructor` (see <a>Custom Type </a>)
- The `interface` type (see <a>Interface</a>)
- The `type function` (see <a>Type Function</a>)
- and `condition types` (see <a>Type Function</a>)

## Defining types

Syntax is

```
// For variables
binding_name [ : <type> ... ] = assignment

// For function
binding_name = [param:<type> ... ] -> <type> block
```

For example

```
// In simple assignment
foo:int = 0

// When declaring function parameters
foo = a:int b:int ->
  // body here

// When declaring the return type of a function
foo = a b -> int
  // body here
```

Some types are "Complex types" eg structures, lists, and potentially custom
types in that they contain other data. For these types, you can specify
containing type by simply adding more `:<type>`

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
=> Error: The type `str` doesn't accept types....
```

## Built-In types

These are the types that are built into the language, while you can "extend"
these you cannot delete these

### Primitives 

The classics

```
int      => integer
float    => float
bool     => boolean
str      => string
```

The functional ones

```
maybe    => Maybe
nothing  => Nothing
just     => Just
```

The one everything defaults to

```
anything => Anything
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

=> Error: The list 'collection' can only have 'int' ....
```

#### Index access

You can access an item in a list via an index as you would suspect

```
collection = [1, 2, 3]

collection[0]
=> 1

collection[2]
=> 3
```

You can also select from the back of a list using negative indices

```
collection = [1, 2, 3]

collection[-1]
=> 3

collection[-2]
=> 2
```

#### Slice and dice

You can also slice up a lists using the colon `:`

Get everything but the first item
```
collection = [1, 2, 3, 4, 5, 6]

slice = collection[1:]
=> [2, 3, 4, 5, 6]
```

Get from `start` to `stop`

```
slice = collection[2:4]
=> [3, 4, 5]
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
```

Grab the last item of a list

```
collection = [1, 2, 3, 4]
x = last collection
=> 1
```

You can get the _head_ and _tail_ of a list easily with destructuring

```
head, ...tail = [1, 2, 3, 4]
=> head 1
=> tail [2, 3, 4]
```


### Structures

Structures in _lang_ are basically JSON

```
data = {
  foo: 'bar',
  num: 123,
}
```

This should create a structure called `data` with fields `foo` and `bar`

You can call a function when declaring a structure and use its value

```
pi -> 3.14

data = {
  foo: 'bar',
  num: 123,
  pi: pi
}
```

This will add a key `pi` with the float value of `3.14` to `data` structure

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

=> data {num: 456, foo: 'bar', pi: 3.14}
```

