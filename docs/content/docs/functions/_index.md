---
title: Functions
---

# Functions

In _lang_ everything is a function and they're everywhere

## Declaring

There's no `keyword` for declaring functions; they're treated like any other
variable. Instead the interpreter will look at the declaration and determine that
the variable you assigned is of the function type.

There are several ways to declare functions based on _arity_ and type specification

tl;dr: syntaxes are

```
function_name -> <body>
faunction_name = <parameter> -> <body>
function_name = ([<parameter>, ...]) -> <body>

where
  <body>      is an expression, or a <type> followed by a newline
              (or semi-colon) and an expression
  <parameter> a binding with optional type ie: "a or a:<type>"
```

### Syntax forms

#### Nullary

The most basic form of function in _lang_ is a function that takes no
parameters. These can be expressed like

```
my_func -> 1 + 2
```

#### Unary

These are functions that only take one parameter. They look like this

```
and_one = a -> a + 1
and_one = a:int -> a + 1
and_one = a:int -> int; a + 1
and_one = a:int -> int
  a + 1
```

pretty straight forward

#### Variadic

Any function takes more than one parameter _lang_ calls _"variadic"_, and they
look like this:

```
add = (a, b) -> a + b
add = (a:int, b:int) -> a + b
add = (a:int, b:int) -> int; a + b
add = (a:int, b:int) -> int
  a + b
```

The difference being the parenthesis; they're required when declaring a
function that has more than 1 parameter

### Returning

Everything in _lang_ is an expression, and so the _return_ is always the last
line of a function:

```
add = (a, b) ->
  a + b
```

but you can also use the `return` keyword if you like

```
add_odds = a b ->
  if a % 2
    return a + b
  a + b
```

useful when breaking out of a function early

### Multiple returns

Technically, functions can't return multple things, they can return a _tuple_ of
things; however the syntax provides sugar to assign multiple variables of a
tuple that's returned

```
name -> 'John', 'Smith'

first, last = name
```

The comma (`,`) makes the return a tuple; in the above case it sets the
return type to a "pair tuple of any" (tuple:any). You can however specify the
types of those as well like any other function, just need some commas in the
right places

```
name -> (str, str)
  'John', 'Smith'
```

## Lambdas

You can also create an anonymous function (as in a function that is not
assigned to a variable), you just have to surround your function with parenthesis

The syntax is:

```
(-> <body>)
(<parameter> -> <body>)
((<parameter>,...) -> <body>)

where
  <body>      is an expression, or a <type> followed by a newline or semi-colon
              and an expression
  <parameter> a binding with optional type ie: "a or a:<type>"
```

For example, a lambda with no parameters

```
dump (-> "Hello World")

=> "Hello World"
```

One with parameters

```
nums = [1, 2, 3]

odds = filter (a -> a % 2) nums

// More than 1 parameter, requires parenthesis
sum = fold ((acc, i) -> acc + i) nums
=> 6
```

You can also have lambdas with multiple retuns by specifying returns with
commas

```
add = (a, b) -> a + b

f = (-> 2, 4)

add (f)

=> 6
```

Sometimes you just need that little extra expression in a lambda and
breaking it out into multiple lines is overkill, for those cases you can ues
the semi-colon `;` to denote multiple expressions

```
some_stuff = [{}, {}, {}]

// A janky way to dump things out without stopping the program
props = stuff.map (i -> dump i; i.prop)
```

## Calling a function

Just as with declaration, calling a function has no ceremony, it's:

```
function_name arg1 arg2 argN ...
```

For example

```
add = (a, b) -> a + b

add 2 2
=> 4
```

You can use parenthesis to be explicit about calling versus passing functions

```
add = (a, b) -> a + b
sub = (a, b) -> a - b

// Explicit
value = add 2 (sub 3 1)
=> 4

// Implicit
value = add 2 sub 3 1
=> Error: The function `sub` wants to 2 parameters, received....
```

### do blocks

Sometimes you want to do run a function as soon as you define it, for that you
can use the `do` syntax

```
fav_food = do
  got = prompt "What's your favorite food? Use commas if needed"
  first (split ',' got)
```

You can specify return types as well

```
fav_food = do:str
  got = prompt "What's your favorite food? Use commas if needed"
  first (split ',' got)
```

with `do` block, every newline denotes a new expression and so if you have
nullary function or sequence of things to run, `do` gives a nice syntax to do
so

### Variadic Parameters

#### Capture

You can define a function that takes `n` number of parameters, by using the
ellipses

```
noop -> ...

// And call it like it expects
noop
=> Nothing

// but you can also call it like this
noop 1 2 3
=> Nothing
```

If you want to assign the parameters to a variable you can do that as well by
adding the variable name to the end of the ellipses

```
numbers = ...nums ->
  // do stuff with list of nums
```

this will capture all the arguments and create a `list:any` assigned to the
`nums` variable in the body. If you want to specify the type you just have to
remember that it's a list of stuff

```
numbers = ...nums:list:int ->
  // do stuff with list of ints
```

#### Spread

You can also "spread out" a list when calling a function by sending each item in
a list to the function as a separate parameter

```
add = (a, b, c) -> a + b + c

stuff = [1, 2, 3]

add ...stuff
=> 6
```

## Overloading

One of the core goals of _lang_ is to let you turn on the strictness as you
need it to allow you to "code jazz" while you're working out an idea but then
later turn up the "type-y-ness" when/if you want

One of the ways to do this is by using the built in pattern matching
functionality to define a function with different arities and return types
(_lang_ calls this "signatures")

For example

```
log_a_thing = f ->
  stdout "Start"
  f
  stdout "End"

log_a_thing = fns:list ->
  stdout "Run list"
  map log_a_thing fns
  stdout "Finish list"

// Then we can call it
log_a_thing (-> stdout 'One')
=> 'Start'
=> 'One'
=> 'End'


log_a_thing [(-> stdout 1), (-> stdout 2), (-> stdout 3)]
=> 'Run list'
=> 'Start'
=> 1
=> 'End'
=> 'Start'
=> 2
=> 'End'
=> 'Start'
=> 3
=> 'End'
=> 'Finish list'
```

The function `log_a_thing` has 2 signatures declared: One where 1 function can
be passed into it and another that accepts a list of functions

When _lang_ sees you define a function with multiple signatures, it will then
assume that you've covered all the signatures you want to accept so if you call
your function with a set of parameters it can't match then you'll get an error:

```
log_a_thing 1 2 3
=> Error: `log_a_thing` expects either 1 parameter....
```

Of course you can workaround this by just adding the "catch all" signature in
your definition:

```
log_a_thing -> ...
  // do something here
```

Now _lang_ will let either 1 function, a list of functions, or anything be
passed into it and run the function that matches the signature

## Functional

### Auto curry / partial application

_lang_ will automatically curry functions if supplied less parameters then expected

```
add = (a, b) -> a + b

add2 = add 2

// Since `add` expected 2 parameters but got 1, it will return a function instead
// of the expression at the end of its body

add2 2
=> 4

add2 3
=> 5

```

You can specify which parameters to partialy apply using the underscore `_` as
well:

```
address = (num, street, country) -> join [num, street, country] ' '

missing_street = address 123 _ 'USA'

complete_address = missing_street 'Main St.'
=> '123 Main St. USA'
```

### Infix

Just about everything in _lang_ is a function, including most operators

For example the operator `+` is actually a function that has a definition like
this

```
add = (a:num, b:num) -> num; a + b

// make the symbol "+" infixable and reference the add function
:infix = ~>dd
```

`infix` is a built-in type that tells _lang_ that a function can be called with
infix notation

Meaning that the following two expressions are equivalent

```
// Called like any other function
+ 1 1

// Called in "infix" position
1 + 1
```

#### Order of operations / precedence

_lang_ is left associative, in order to make it not so, you have to use
parenthesis

```
// The following is using the `divide` and `plus` function in infix position
4 / 4 + 4
=> 5

// You could be explicit if you liked to
4 / (4 + 4)
=> 0.5

// without infix
+ 4 (/ 4 4)
=> 5
```
