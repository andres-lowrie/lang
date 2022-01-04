---
title: Functions
---

# Functions

In _lang_ *everything is a function

## Declaring

There's no `keyword` for declaring functions; they're treated like any other
variable. Instead the interpreter will look at the declaration and determine that
the variable you assigned is of the function type.

Abstractly functions look like this

```
// Without types
the_name_of_the_function = (parameters) -> <body>

// With types
the_name_of_the_function:rtype = (p1:t1, p2:t2, ...) -> <body>

// If signature gets too long for your taste
// you can break up into multiple lines
// where a <tab> delimits each level of the function
the_name_of_the_function:rtype = (
<tab>p1:t1,
<tab>p2:t2) ->
<tab><tab>body
```

### Syntax forms

#### Nullary

The most basic form of function in _lang_ is a function that takes no
parameters. These can be expressed like

```
my_func = () -> 1 + 2
```

However, when a function takes no parameters, then the `=` can be omitted and the _thin arrow_ alone can be used.

```
// Both of these mean the same thing
my_func = () -> 1 + 2
my_func -> 1 + 2
```

You can express the return type of a function the same way you would for any other variable:
```
my_func:num -> 1+2
```


#### 1 or more parameters

_parenthesis_ are **required** if the function takes one or more parameters

```
add = (a, b) -> a + b
add = (a:int, b:int) -> a + b
add:int = (a:int, b:int) -> a + b
```

#### Multiple Line Declaration

A function can span multiple lines but the body of the function must be intended, ie: _lang_ is whitespace sensitive for multiple lined functions:

```
log_and_add = (a, b) ->
  log "Adding", a, "+", b
  a + b
```

[Read more about the way _lang_ should be formatted]()


### Returning

Everything in _lang_ is an expression, and so the _return_ is always the last
line of a function:

```
add = (a, b) ->
  a + b
```

You can break out early by calling the `return` function in the body of a function

```
add_odds = (a, b) ->
  if a % 2
    return a + b
  a + b
```

[read more about the return macro function]()

### Type terseness

Sometimes you have functions that take a bunch of the same type of parameter. Instead of having to repeat those types over and over again, you can declare multiple parameters to be of the same type using the comma. _lang_ calls these "_type groupings_"

```
// Both of these are equivalent
volume = (a:num, b:num, c:num) -> a * b *c
volume = (a,b,c:num) -> a * b *c
```

Note that you're not limited to one group, you can use as many as you like
```
user_info = (fname,lname:str, phone:num, mail_address:email) ->
```

**Note**: Type groups must not include a space after the comma, since the space and comma combined is how _lang_ parses different parameters.

In other words

```
// This will parse as
fn = (a, b:int)
=> a:any 
=> b:int

// while this will parse as
fn = (a,b:int)
=> a:int 
=> b:int
```

When the return type and all the parameters are of the same type, there's sugar for that too
```
// Both of these are equivalent
volume:num = (a:num, b:num, c:num) -> a * b *c
volume = :num: (a, b, c) -> a * b *c

```
Here the `:<type>:` syntax is saying:

> :Input is num and  Output: is num

_lang_ calls this function the `tio` function for "type in/out"

[Read more about the `tio` function]()

## Applying / Calling a function

Just as with declaration, calling a function has no ceremony, parameters are denoted by commas

```
function_name arg1, arg2, argN, ...
```

For example

```
add = (a, b) -> a + b

add 2, 2
=> 4
```

You can use parenthesis to be explicit about calling versus passing functions

```
add = (a, b) -> a + b
sub = (a, b) -> a - b

// Explicit
value = add 2 (sub 3, 1)
=> 4

// Without the parenthesis _lang_ breaks
value = add 2, sub 3, 1
=> Compile error: The function `add` wants to 2 parameters, received....
```

### Multiple line application

_lang_ is whitespace sensitive since it does some clever parsing to favor expressiveness. One of these parsing considerations is to allow for applying a function over parameters that are spread out over multiple lines.

Say for example you have a function that takes a whole bunch of parameters

```
my_func = (p1, p2, p3, p4, p5, p6) -> // implementation
```

We _could_ call it like this:
```
my_func 1, 2, 3, 4, 5, 6
```

but if those values were longer or had complex variable names then that would become very hard to read. 

The other way to call the function would be to place each parameter in its own line and omit the comma; not omit persay instead we swap out the comma for a `<newline>`
```
my_func
  1
  2
  3
  4
  5
  6
```

> This comes in handy when dealing with function that take other functions as control strucutres, for example the `if`. [Read more about the `if` function]()


## Multiple returns

Technically, functions can't return multple things, they can return a _tuple_ of things; however the syntax provides sugar to assign multiple variables from the tuple that's returned

```
name -> 'John', 'Smith'

// Long way
tmp = name
fname = tmp[0]
lname = tmp[2]

// Sugar
fname, lname = name
```

When the value of the assignment can fit into 1 line, then the comma (`,`) can be used to express the tuple. If the value function needs to span multiple lines however, then you need to call the `tup` function directly

```
name -> tup
  'John',
  'Smith'
```

```
// All of these return the same tuple
name -> 'John', 'Smith'
name -> tup 'John', 'Smith'
name -> tup
  'John',
  'Smith'

// with types
name:tup:str,str -> 'John', 'Smith'
name:tup[str,str] -> 'John', 'Smith'
```

## Lambdas

You can also create an anonymous function (as in a function that is not
assigned to a variable)

The syntax is

```
() -> <body>
(<parameter>) -> <body>
(<parameter>,...) -> <body>
(<parameter>,...):<type> -> <body>

where
  <body>      is an expression, or a <type> followed by a newline or semi-colon
              and an expression
  <parameter> a binding with optional type ie: "a or a:<type>"
```

For example, a lambda with no parameters

```
typeof () -> "Hello World"

=> fn:str
```


One with parameters

```
nums = [1, 2, 3]

// single
odds = filter (a) -> a % 2, nums

// multiple
sum = fold (acc, i) -> acc + i, nums
=> 6
```

When specifying the return type of a lambda, the `:type` is moved to the right of the parenthesis
```
nums = [1, 2, 3]

odds = filter (a):bool -> a % 2, nums
```

When returning a tuple from a lambda, you need to either specify the return type or call the `tup` function

```
stuff = [...]

// Both of these are equivalent
apply ((a):tup -> 2, 4), stuff
apply (a) -> tup 2, 4, stuff
```

Note that most of the time, either because of type inference or manual declaration, adding a return type to a lambda is usually redundant [read more about leveraging types]().

Sometimes you just need that little extra expression in a lambda and breaking it out into multiple lines is overkill, for those cases you can use the semi-colon `;` to denote multiple expressions

```
stuff = [{}, {}, {}]

// A janky way to dump things out without stopping the program
props = map (i) -> dump i; i.prop, stuff
```


## Variadic / Capturing Parameters

You can define a function that takes different groupings of parameters by using the ellipses (`...`), _lang_ calls this capturing.

For example this function takes any arguments passed to it and prints them out

```
parrot = (...args) ->
  apply print, args
```

What `...<name>` does is take *all the parameters and create a variable with the name `<name>` that is scoped to the function body. Like any other variable binding the type is inferred but you can also specify a type by using a colon, the thing to remember that the type needs to be a `list:<type>`

```
parrot = ...args:list:str ->
  apply concat, args
  
// It strictly has to be the list type
// however
parrot = ...args:iter:str ->
  apply concat, args
  
=> Compile error: The function `parrot` specified an incorrect type for ...
```

Capturing isn't limited to "all or nothing", you can also use it at the end of the parameter definition

```
foo = (a, ...rest) ->
  do_something a
  process ...rest
```

You can use it at the front, read as "capturing everything except the last parameter"

```
personal_info = (...names, email) ->
```

Or in the middle, read as "capture everything except the first and the last parameter"
```
graph_path = (start, ...vists, end) ->
```


## Spread

You can also "spread out" a collection of things when calling a function by sending each item in a list to the function as a separate parameter

```
add = (a, b, c) -> a + b + c

stuff = [1, 2, 3]

add ...stuff
=> 6
```

### Collections

Note that the spread syntax isn't limited to just lists, any [collection]() type can actually be spread out, for example:
```
sum = (a, b) -> a + b

// A tuple
pair = 1, 2
sum ...pair

// A structure
pair = {a: 1, b: 2}
sum ...pair
```

> **Note**: there's more going on with structure spreading in the last example, read about it [here]()


## Overloading

One of the core goals of _lang_ is to allow you to "code jazz" while you're
working out an idea but then later turn up the "type-y-ness" when/if you want it

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
  apply log_a_thing, fns
  stdout "Finish list"

// Now we can start using it
log_a_thing () -> stdout 'One'
=> 'Start'
=> 'One'
=> 'End'


log_a_thing [() -> stdout 1, () -> stdout 2, () -> stdout 3]
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
be passed into it and another one that accepts a list of functions

When _lang_ sees you define a function with multiple signatures, it will then
assume that you've covered all the signatures you want to accept so if you call
your function with a set of parameters it can't match then you'll get an error:

```
log_a_thing 1 2 3
=> Compile error: `log_a_thing` expects either 1 parameter....
```

## Functional

### Auto curry

_lang_ will automatically curry functions if supplied less parameters then expected

```
add = (a, b) -> a + b

add2 = add 2

// Since `add` expected 2 parameters but got 1, it will return a function

add2 2
=> 4

add2 3
=> 5
```

When _lang_ currys a function, the function that was returned will accept all the parameters that weren't supplied originally, eg:

```
volume = (a, b, c) -> a * b * c

step = volume 12

final = step 6, 3
```

### Partial Application

Unlike with currying, with partial application you can specify which parameters to partialy apply using the underscore `_`:

```
address = (num, street, country) -> join [num, street, country]

missing_street = address 123, _, 'USA'

complete_address = missing_street 'Main St.'
=> '123 Main St. USA'
```

You can denote any number of parameters as "missing" via the `_` just as long as you pass at least one:

```
address = (num, street, country) -> join [num, street, country]

missing_street_country = address 123, _, _

country = address _, _, 'MEX'

// But the following won't work however
missing_everything = address _, _, _
=> Compile Error: `address` expects 3 parameters but all were skipped. This...
```

### Infix

In _lang_ everything is a function there are no operators.

For example the operator `+` is actually a function that has a definition like
this

```
add = :num: (a, b) -> a + b

// make the symbol "+" infixable and reference the add function
+:infix = add
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
