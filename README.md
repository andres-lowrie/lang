language
========

## It should

- Be ultra configurable and extensible at **every** point
- Be declarative over imperative (functional)
- Have commands and the cli be first class citizens
- Be as strict as you make it but guess everywhere else
- Be good at guessing (perhaps ... magical?)
- Allow testing to be a first class citizen while still letting the code say what
  it wants to say (tests should bend to code not the other way around)
- Stuff should just be included

# Syntax

## Primitive Types

### Numbers

They should work as expected and the compiler should do the work of figuring out
what type you intended

```
1         => Integer
1.4       => Float
1_000_000 => Integer
1/3       => Float

// You should always have the final say though
50:double => Double
```

### String


Single, Double, or use back-ticks for literal

```
"hello"         => "hello"
'hello "world"' => 'hello "world"'
`hello 
world`          => 'hello \n world'
```

When dealing with shell commands however, strings become a huge pain point; The
core library should have functions to help with this


```
// From the explicit ones
escapeDoubleQuote 'No "double quote please"'
-> 'No \"double quote please\"

// To ones that guess
escape "Escape 'double quote'"
-> "Escape \'double quote\'"

escape `Some string that's a 'mix' of "strings"`
-> `Some string that\'s a \'mix\' of \"strings\"`
```

### Booleans

```
true => true
false => false
```

## Complex Types

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

## Types

### Declaring

Sometimes you want to define the type of things that are in a structure, for
that you can use the `type` function the syntax is:

```
struct = type {
  field_name: type
  ...
}
```

For example

```
person = type {
  first: string
}

// or in one line if you like
person = type { first: string }
```

You can also declare a type and assign defaults to the fields

```
person = type {
  first:string = 'Jane'
}
```

Now whenever a `person` is passed around, it will always have a field `first`
and the value will always be 'Jane'

In other words when assigning values to a `type`, those values are immutable.

### Instantiating

When you declare a `type` you're actually declaring a function.

When you do

```
person = type { first: string }
```

You're declaring a function named `person` that takes a [type block]() as its only
parameter. Meaning that to "instantiate" a person, you call the function just
like any other function.

```
// non strict
developer = person { 'Jane' }

// or strict
developer = person { first:'Jane' }
```

When using the non-strict syntax the order of the parameters will be assigned in
the same order the fields of the structure where declared in other words

```
abc = type { a:int, b:int, c:int }
easy_as = abc 1 2 3

=> abc { a:1, b:2, c:3 }
```

#### Functions everywhere (macros?)

You can leverage the functional nature of _lang_ to create types on the fly as
well

```
getData -> {
  first = prompt "What's your name?"
  last = prompt "What's your last name?"
  {first, last}
}

user = type getData
stdout user

// => "What's your name?"
// > Bobby
//
// => "What's your last name?"
// > Sue
//
// => user { first:'Bobby', last: 'Sue' }
```

#### Mutability

Structures are basically just `types` but you're declaring and instantiating
them at the same time. 

So when you do 

```
person = {
  name: 'Jane',
  last: 'Doe'
}
```

You're actually doing this

```
person = mut type {
  name: string,
  last: string,
} { name: 'Jane', last: 'Doe'}

// => person { name: 'Jane', last: 'Doe' }
```

The difference between a structure and a type is the fact that the
structure function will return a type that is _mutable_ and the type function
returns one that isn't

Which is what that `mut` function is doing there in last example

#### ... and that let's you control strictness at the time you want to control it

```
// At this point we don't know what we want to lock down
// so we'll leave it open
data = { 
  truth: 4
}

propaganda = prompt "Share truth? 2 + 2 = " + data.truth

// => Share truth? 2 + 2 = 4
//
// > 5

data.truth = propaganda

// Now we want to lock it down, so we can easily do that
// from this point on in the program
data = type data

// thus we can modify the type anymore
data.truth = 4

=> Compile error: The type `data` shouldn't have its field `truth` ....
```

### Self referencing

Sometimes you want to access the thing you're declaring while you're declaring
it, so _lang_ should have support for that

```
data = type { 
  first: string,
  last: string,
  name: (-> data.first + data.last)
}

// Later

d = data { 'Jane', 'Doe'}
d.name

=> 'Jane Doe'
```

## Variables

### Basics

Very little ceremony when it comes to variable assigning

```
foo = 1             => foo is the number 1
foo = 'something'   => foo is the string 'something'
foo = { bar: 'bas'} => foo is a type
foo = a b -> a + b  => foo is a function
```

Basically it's 

```
anything = anything
```

_lang_ will infer types for you but you tell it what they should be if you like

```
foo:int = 42
```

If you want to declare variables and not assign them a value... you're allowed
to do that ...

```
foo:int
```

but why would you want to?...that's the real question

### Destructuring

You can also destructure types and functions

#### Structures

When pulling out variables from a structure, you need to name the variables the
same thing they're named in the structure

```
data = {a: 'a', b: 'b'}

// Infer type
a, b = data

// Strict type
a:string, b:string = data

// The following will not work
a, c = data

// => Runtime error: The type `data` doesn't have a field `c` ....

```

#### Lists

When destructuring lists however, you can name the variables anything you want

```
data = [1, 2, 3]

// Infer type
a, b, c = data

// Explicit type
a:int, b:int, c:int = data
```

You can also either ignore the things in the list you don't care about or
capture them as a list in a new varialbe

```
data = [1, 2, 3, 4, 5, 6, 7]

a, b , c, ...rest = data

// => a 1
// => b 1
// => c 1
// => rest [4, 5, 6, 7]
```

#### Functions

If a function returns multiple things you can assign results to
multiple variables

```
nautical -> 'port', 'starboard'

left, right = nautical

// => left 'port'
// => right 'starboard'
```


## Functions

### Just a variable

There's no `keyword` for declaring functions; they're treated like any other
variable. Instead the compiler will look at the declaration and determine that
the variable you assigned is of the function type.

The syntax is

```
function_name = parameters -> body
function_name -> body
```

### Basics

#### The thin arrow

There isn't a whole bunch of ceremony to declaring functions: the only thing you
really need is the _thin_ arrow and the assignment operator

```
add = a b -> {
  a + b
}
```

This tells the compiler that the left side of the arrows are parameters and the
right side of the arrow is the body.

The return is implicit as in the last expression will be returned, but you can
also use the `return` keyword if you like

```
add = a b -> {
  return a + b
}
```

#### Types

You can crank up the strictness as well

```
// Explicit return but lax parameters
add = a b -> int {
  a + b
}

// Explicit parameters but lax return
add = a:int b:int -> a + b

// Full explicitness
add = a:int b:int -> int {
   a + b
}
```

#### Multiple returns

Functions can return multiple things instead of just one thing.
To do this, you have to use the 

```
name -> {
  'John, 'Smith'
}

first, last = name
```

You can specify the types of those as well like any other function just need
some commas in the right places

```
name -> string, string {
  'John', 'Smith'
}
```

#### Shorthand

Ergonomics are important

You can declare functions that don't take any parameters:

```
add = -> {}

// But that equal sign looks weird, so you can leave it out
add -> {}
```

The _brackets_ are completly optional when not specifying a return type:

```
// No params
show_a -> 'a'

// Some params
add = a b -> a + b

// Sometimes you just need that little extra 
add = user addr perms ->
  user + addr + perms


// I mean this is technically valid but don't do this
add = a b c d e f g h i j ->
  a + b + c + d + e + f + g
  + h + i + j
```

Functions with multiple retuns can also be one liners

```
yes_no = true, false
```

The compiler will create a [body block]() around anything on the right hand side
of the thin arrow if you don't specify one.

The only time you're required to specify one (body block) is when you want to be
explicit about the return of a function

#### Lambdas

You can also create an anonymous function (as in a function that is not
assigned to a variable), you just have to surround your function with parenthesis

```
say = something -> print something

say (-> "Hello World")
```

If your lambda needs params, then you should be able to define them the same way
you do in any other function

```
nums = [1, 2, 3]

sum = fold (acc i -> acc + i) nums
// => 6
```

You can also have lambdas with multiple retuns

```
add a b -> a + b

add (-> 2, 4)

// => 4
```

### Calling a function

Just as with declaration, calling a function has no ceremony it's:

```
function_name arg1 arg2 argN ...
```

For example

```
add a b -> a + b

add 2 2
=> 4
```

By default _lang_ will call your function when it appears anywhere (except when
it's being declared, or passed as parameters)

### Parameters

By default _lang_ will allow you to pass anything into a function regardless of
the "arity" of your function

For example

```
// You can have a function that looks like 
noop -> {}

// And call it like it's defined
noop
// => Nothing

// but you can also call it like this
noop 1 2 3
// => Nothing
```

#### Auto "rest" params

By default, _lang_ will accept any _extra_ parameters passed into a
function and assign them to the [list]() variable `args` in your function body

```
add -> {
  stdout args
}

add 1 2 3 4
=> [1, 2, 3, 4]
```

If you want to assign another name to `args` you can do either it via [config]()
or when you declare your function using the variadic syntax

```
add ...nums -> {
  // The compiler will assign args to nums in this case
  // nums = args
}
```

You can also declare function that take both positional and variadic parameters

```
add = a b ...others -> {
  // do something
}
```

Of course you can increase the strictness and specifically say "no, you take
no parameters dammit"

```
coinfig.no.rest_params {
  add = () -> {}
}
```

#### Spread

You can also "spread out" a list when calling a function by sending each item in
a list to the function as a separate parameter

```
add a b c -> a + b + c

stuff = [1, 2, 3 ]

add ...stuff
=> 6
```

### Functional

At it's core, _lang_ is a functional language meaning that certain features
available to other functional languages are baked in

#### First-class citizens

Pass them anywhere

As a variable

```
add = a b -> a + b

n -> rand 1 10

add n 4
```

Return them in other functions

```
add = a b -> a + b
sub = a b -> a - b

add2Sub3 = a -> compose (add 2) (sub 3)

add2Sub3 1
=> 0
```

On the fly

```
add = a b -> a + b

add (rand 1 10) 4
```


#### Auto curry / partial application

Just as how when you pass [extra]() parameters to a function _lang_ does you a
solid and presents them to you in your function

If you pass a function "less" parameters than it expected, _lang_ will
automatically create a partially applied function

```
add = a b -> a + b

add2 = add 2

// Since add expected 2 parameters but got 1, it will return a function instead
// of the expression at the end of its body

add2 2
=> 4

add2 3
=> 5

```

Like 9 out of 10 times this is great and you want this, but every now an then
you don't want to auto curry; you can disable it ad-hoc on the fly via a [scope block]()

```
add = a b -> a + b

// Temporarily disable auto-curry
config.no.auto_curry {
  add2 add 2
}

=> Runtime error: add wanted 2 parameters, received only 1 on line ....
```

##### What does this mean for multiple return?

Like everything else, you can disable auto curry via the [config]() if you like
... but you don't want to do that. :wink:



## Blocks

These are the most magical pieces of _lang_ since this syntax is used everywhere
and the compiler uses context to determine types

The synax is:

```
{ ...grammar.expression }
```

#### Forms of blocks...down the rabbit hole

They're 2 forms of blocks. [body]() blocks like the kind you see in function
declarations

```
fn -> { // body block}
```

and [type]() blocks like those you see in type declarations

```
t = type { field: type}
```

Context is what determines the different forms of blocks. _lang_ will default to
_body block_ when the context is ambiguous

For example

```
{
  x:int = 42
}
```

This could be "create a type with a field named
_x_ of type int and a value of 42" or it could be "declare a variable _x_ of
type _int_ with a value of 42".

Because there's not enough context to determine what's happening with the block
(there's no prefixed function). _lang_ will assume you just want to create a
body block to scope variables in this case.

This is the case if there's no assigment as well:

```
{
  x:int
}
```

This is just as ambiguous as the one with the assignment

#### The magical comma

When the compiler sees a comma after an expression followed by another
expression (as long as it's not the last expression) then it's a safe bet that a
_type block_ is being declared

```
{
  x:int,
  y:int,
  z:int,
}
```

The compiler will classify that as a type block because body blocks don't have
commas as expression terminators

But if the last expression has a comma in it, then it's a body bock since this
means that the function is returning multiple things

```
{
  x:int,
  y:int,
  z:int, all:point //<= That means this is a function and therefore a body block
}
```

that one **is** a body block


If we remove the commas, then _lang_ will think you're just declaring variables
and pick body block

```
{
  x:int
  y:int
  z:int
}
```


#### The assignment operator

Using the assignment operator `=` in a body can also provide context to the
compiler.

A type block can only have the assignment operator in one place and that's after
type declaration on a field some. So if the assignment operator appears before
the colon in an expression, the the compiler will determine that we're wanting a
body block

```
// Body block
{
  x = 42
}
```

Once the compiler makes a decision on the type of block in play, you can't
change it later

```
x = {
  // Ambiguous
  x:int = 44,
  y:string,
  foo:'bar',

  // The following line will tell the compiler that this is a body block
  y = 'something else',
}

So if you try to pass it as a type block _lang_ will yell at you

y = type x

// Compile error: Incorrect block type. The function `type` doesn't allow ....
```

#### Lock it down

If you pass a block to a function that declares what type of block it wants,
then _lang_ won't do any guessing and instead do compile checking

For example here's the [type]() function declaration

```
type = grammer.block:type -> type {
  // implementation
}
```

This tells the compiler that the `type` function can only ever receive a type
block and thus when the compiler parses code that uses it, it will only ever
look for type blocks and clear up ambiguous code

```
// This is okay
data = type { 
  x: 32
}

// This is okay as well
data = type { 
  x:int = 32
}

// This is not okay
data = type { 
  x:int,
  y = 4
}

// Compile error: Incorrect block type. The function `type` doesn't allow ....
```

You can be just as explicit for body blocks if you like by using the body function

```
greet = body { 
  x:string = 'hello'
  y = 'world'
  
  x + y
}

stdout greet

=> "hello world"
```
