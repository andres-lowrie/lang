# Types

Type is an overloaded term in _lang_.

When the word "type" is used it is in reference to 5 things:

- The `data types` aka "Built In Types" (ie: Integer, Boolean, etc.)
- The `type constructor` (see <a>Custom Type </a>)
- The `interface` type (see <a>Interface</a>)
- `condition types` (see <a>Conditional Types</a>)
- and `dynamic types` (see <a>Dynamic Types</a>)

## Built-In types

These are the types that are built into the language, while you can "extend"
these you cannot delete these

### Primitives 

The classics

```
int      => Integer (32 or 64 bit depends on target)
float    => Float (64 bit float)
bool     => Boolean
str      => String
```

The functional ones

```
maybe    => Maybe
nothing  => Nothing
just     => Just
```

The collection 

```
list   => List
struct => Structure
tup    => Tuple
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

You can have mixed stuff <sub>(technically, this becomes of `list:Anything`, but who cares about that)</sub>

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

_lang_ has some built in functions that make index access more sugary, specifically

`head`  => returns first item in a list
`tail`  => a list comprised of all the elements except the first
`first` => an alias for `head`
`last`  => returns last element

```
collection = [1, 2, 3]

dump head collection
=> 1

dump tail collection
=> [2, 3]

dump first collection
=> 1

dump last collection
=> 3
```

#### Slice and dice

You can also slice up lists using the colon `:`

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

You can also define inifinte ranges but you have remember that it's lazy
```
collection = [1..]

dump head collection
=> 1

// but it's lazy so the following will transform your laptop into a space
// heater
dump collection
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

=> data {num: 456, foo: 'bar', pi: 3.14 }
```

This will add a key `pi` with the float value of `3.14` to `data` structure


#### Shorthand

You can pass in variables and the structure will be assigned key:value pairs
using the variable name and value

```
foo = 'bar'
pi -> 3.14

// Declaring it like this
data = {
  num: 456,
  foo,
  pi
}

=> data {num: 456, foo: 'bar', pi: 3.14}
```

### Tuples

Immutable collection of stuff

```
my_tup = (1,2,3)
```

You can also define what kind of stuff it is

```
my_tup:tup:int = (1, 2, 3)
```

## Defining types

Declaring something of a type

```
// For variables
binding_name [ : <type> ... ] = assignment

// For functions
binding_name = [(param:<type> ... )] -> <type> block
```

For example

```
// In simple assignment
foo:int = 0

// When declaring function parameters but not the return type you can use the
// single line form
foo = (a:int, b:int) -> body here

// When declaring the return type of a function, you have to use a new line
// since that's how _lang_ determines the expression block...
// also it's easier to read imho
foo = (a, b) -> int
  body here
```

### Type Chaining

Some types are "Complex types" eg structures, lists, and potentially custom
types in that they contain other data. For these types, you can specify
containing type by simply adding more `:<type>`

A way to think of it is that the colon `:` can be read as _"which is of type"_


```
// A list that holds strings
//
// foo "which is of type" list "which is of type" str
foo:list:str = [...]

// foo is a list ... that holds lists.... that holds strings
foo:list:list:str = [[]...]
```

In other languages the last one could look like this maybe

```
Foo<T: Iterator<T: Iterator<String>>>
```

in _lang_ you just spam the colon key instead.


The "type chaining" syntax works as long as the thing on the left of the `:`
can accept a type so for example the following wouldn't work

```
foo:str:list = ...
=> Compile Error: The type `str` doesn't accept types....
```

### Type Aliases

Sometimes you want to be even more explicit defining what things can be inside
of other things.

Here's a contrived example where you're calling an unrealible service
(contrived because that never happens in the wild :wink:)

```
foo:list:list:maybe:structue = [[{},{},{}], [{},{},{}],...]
```

the type definition is gnarly af, so we can make an alias instead to keep the
line easy to read

```
// plot twist, an alias is just a regular assignment
janky_svc = maybe:structue
janky_results = list:janky_svc
bulk_results = list:janky_results

foo:bulk_results = [[{},{},{}], [{},{},{}],...]
```

Type aliases allow the code to better express the intent of a structure by
allowing us to break it up into domain specific names
