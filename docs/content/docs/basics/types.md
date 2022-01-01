# Types

Type is an overloaded term in _lang_.

When the word "type" is used it is in reference to several things given the context:

- The `data type` which can be _primitives_ or _structures_ (see [link to core constructs]())
- The `type constructor` (see [Custom Type]())
- The `interface` (see [Interface]())
- `condition types` (see [Conditional Types]())

_"Type Checking"_ also happens at different times in _lang_, for example data types primitive types can be checked at compile types, but condition types can only be checked at run time. When it comes to _lang_, _types_ can be better described as _constraints_ as opposed to "traditional type" systems

None the less we'll use the term "type" throughout and procide link to in-depth docs for technical details

## Built-In types

These are the types that are built into the language. All other types build off of these types in some form or fashion

### Basic

Caveman style

```
int      => Depends on target
int8
int16
int32
int64
uint8
uint16
uint32
uint64

float
float..  => the same for float

char     => utf8 by default
```

The classics

```
bool     => Boolean
str      => String
enum     => Enumerable
```


The interface ones

```
num      => Number
iter     => Iterable
```

The functional ones

```
maybe    => Maybe
nothing  => Nothing
just     => Just
either   => Either
optional => Optional
```

The collection ones

```
list   => List
struct => Structure
tup    => Tuple
```

The one everything defaults to

```
any => Anything
```

### Lists

A list can hold stuff.

You can have the same stuff

```
collection = [1, 2, 3]
```

You can have mixed stuff <sub>(technically, this is interpreted to be `list:Anything`, but who cares about that)</sub>

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

head collection
=> 1

tail collection
=> [2, 3]

first collection
=> 1

last collection
=> 3
```

#### Slice and dice

You can also slice up lists using the hyphen `-`

Get everything but the first item
```
collection = [1, 2, 3, 4, 5, 6]

slice = collection[1-]
=> [2, 3, 4, 5, 6]
```

Get everything until the 4th item
```
slice = collection[0-3]
```

or from `start` to `stop`
```
slice = collection[2-4]
=> [3, 4, 5]
```

#### Ranges

You can also define lists via a ranage syntax

```
collection = [1..99]
=> [1, 2, 3 .. 99]
```

You can also define inifinte ranges but you have to remember that the range syntax will return a [lazy]() list
```
collection = [1..]

head collection
=> 1

// so the following will transform your laptop into a space heater
tail collection
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

You can also point to variables as values:

```
pi = 3.14

data = {
  foo: 'bar',
  num: 123,
  pi: pi
}

=> data
=> { num: 456, foo: 'bar', pi: 3.14 }
```

This will add a key `pi` with the value from the variable named "pi"


#### Shorthand

If you pass in variables the structure will be assigned key:value pairs
using the variable name and its value

```
foo = 'bar'
pi = 3.14

// Declaring it like this
data = {
  num: 456,
  foo,
  pi
}

=> data
=> { num: 456, foo: 'bar', pi: 3.14 }
```

#### Dynamic keys

You can also define a `key` name dynamically as well

```
foo = 'my_key'

// Declaring it like this
data = {
  '{foo}': 123
}

=> data { my_key: 123 }
```

This leverages the fact that all strings can be _interpolated_ and uses the
"_braces_" to do so at runtime. (see [runtime declarations] for details())

**umm actually**


The keys for a structure aren't limited to just strings, they can be anything.

For examples numbers:

```
my_struct = {
  1: 'one',
  2: 'two',
  3: 'three'
}
```


or lists

```
my_struct = {
  [1,2] = ["one", "two"],
  ...
}
```

of functions ... of any type

```
my_struct = {
  int: "the key is an integer",
  str: "the key is a string",
  ...
}
```

of the results of functions

```
my_struct = {
  (env.user): 'Value of $USER',
  (env.home): 'Value of $HOME,
}

=> { janesmith: 'Value of $USER', /home/janesmith: 'Value of $HOME }
```


### Tuples

Immutable collection of stuff.

```
my_stuff = 1,2,3
```

You can also define what kind of stuff it is

```
my_stuff:tup:int = 1, 2, 3
```

you can also mix and match
```
my_stuff = 1, "two"
```

If your tuple needs to span multiple lines, then you have to call the function explicitly
```
my_stuff = tup
  "one",
  "two",
  "three", 
  "four",
  "five",
  "something long requiring you to break the function call into multiple lines"
  
```

[read more on immutability]()

[read more on multiple line function calls]()


## Defining types

Declaring something of a type just requies usage of the colon (`:`)

For example

```
// In simple assignment
foo:int = 0

// In a one-line function where the parameters have types defined but not the
// return type
foo = (a:int, b:int) -> ...

// and a one liner with a return type
foo = (a:int, b:int):int -> ...
```

More info in [functions]()

### Type Chaining / Container Types

Some types are "Complex types" eg structures, lists, and potentially custom
types in that they contain other data.

For these types, you can specify the contained type by simply adding more
"`:<type>`" as needed, _lang_ calls this practive _"type chaining"_

A way to think of it, is that the colon "`:`" can be read as _"of type"_ and when it
appears after a type that can consist of other types it's read as "consisting
of type"


```
// A list that holds strings
//
// foo "of type" list "consisting of" str
foo:list:str = [...]

// foo is a list ... that holds lists.... that holds strings
// or
// foo "of type" list "consisting of type" list "consisting of type" str
foo:list:list:str = [[]...]
```

When the containing type has a specifc set of members (like for example an
`enum` or a `tup`) then you denote the members with a comma:
them with commas:

```
// foo "of type" tup "consisting of a str and a str"
foo:tup:str,str = 1, 2

// This allows for complex types to be expressed in a more readable way
// for example

foo:iter:iter:str = [["some"], ["strings"]]
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


**umm... actually**

Sometimes you can't just easily chain types if the type takes `n` number of _mixed_ members

A contrived example 

Suppose we want to declare a list of tuples
where the list can only contain 2 tuples
and each tuple can contain only 2 members of different types

If we try and use straight up type chaining
```
foo:list:tup:int,int,tup:float,float = 1, 2
```

_lang_ will get lost when when it sees the second `tup` in the signature because the tuple type can contain any number of elements, including other types, so instead of what we want we'll actually get back:

> a list

> containing a tuple

> which can contain 2 ints and a tuple

> where the last tuple can contain 2 floats

To make that last type work, you can use brackets to group contained members, so it becomes
```
foo:list:tup:[int,int],tup:[float,float] = 1, 2

// or even more explicit
foo:list:[tup:[int,int],tup:[float,float]] = 1, 2
```


While grouping is allowed for very complex types, the _lang_ way of doing things is to use _type aliases_ instead of inlining very hard to read complex container type

### Type Aliases

Sometimes you want to be even more explicit defining what things can be inside
of other things.

Here's a contrived example where you're calling an unrealible service
(contrived because that never happens in the wild :wink:)

```
foo:list:list:maybe:structue = [[{},{},{}], [{},{},{}],...]
```

the type definition is gnarly af, so we can make aliases instead, that will keep the
signature line easy to read

```
// plot twist, a type alias is just a regular assignment

janky_svc = maybe:structue
janky_results = list:janky_svc
bulk_results = list:janky_results

foo:bulk_results = [[{},{},{}], [{},{},{}],...]
```

Type aliases allow the code to better express the intent of a structure by
allowing us to break it up into domain specific names, assuming that each type can be resolved at compile time, the parser will actually take these aliases and "inline" them.

[read more about compile v run time type enforcement]()
