---
title: Functions
---

# Functions

In _lang_ everything is a function and they're everywhere

## Declaring

There's no `keyword` for declaring functions; they're treated like any other
variable. Instead the compiler will look at the declaration and determine that
the variable you assigned is of the function type.

The syntax is

```
function_name = parameters ... -> body
function_name -> body
```

### The thin arrow

There isn't a whole bunch of ceremony to declaring functions: the only thing you
really need is the _thin arrow_ and an assignment binding

```
add = a b -> {
  a + b
}
```

This tells the compiler that the left side of the arrow are parameters and the
right side of the arrow is the body.

a function can be declared to take 0 or any number of parameters

```
// None
add -> {}

// 1
add = a -> {}

// Many
add = a b -> {}
add = a b c -> {}

// 3 defined, and the rest into a list
// _lang_ calls this "variadic params"
add = a b c ...d -> {}
```

More info on <a ref="#spread">variadic params</a>


### Returning

The return is implicit as in the last expression will be returned,

```
add = a b -> {
  a + b
}
```

but you can also use the `return` keyword if you like

```
add = a b -> {
  return a + b
}
```

### Types

You can crank up the strictness as well

```
// Strict return but lax parameters
add = a b -> int {
  a + b
}

// Strict parameters but lax return
add = a:int b:int -> a + b

// Full strictness
add = a:int b:int -> int {
   return a + b
}
```

### Multiple returns

Functions can return multiple things instead of just one thing.

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

## Shorthand

Ergonomics are important

You only need the `=` operator when your function takes parameters.

```
// Takes no parameters so no `=` is needed
give_a -> { "a" }
```

The _braces_ are completly optional when not specifying a return type:

```
give_a -> 'a'
```

You can loose the braces all together if you want

```
// No params
give_a -> 'a'

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

You can use shorthand form for functions with multiple retuns as well:

```
yes_no -> true, false
```

By default, the compiler will create a <a>block</a> around anything on the
right hand side of the thin arrow if you don't specify one.

The only time you're required to specify one (a block) is when you want to be
explicit about the return of a function

## Lambdas

You can also create an anonymous function (as in a function that is not
assigned to a variable), you just have to surround your function with parenthesis

The syntax is:

```
([params] -> body)
```

where `params` are optional.

For example

```
say = something -> stdout something

say (-> "Hello World")
```

If your lambda needs params, then you should be able to define them the same way
you do in any other function

```
nums = [1, 2, 3]

sum = fold (acc i -> acc + i) nums
=> 6
```

You can also have lambdas with multiple retuns

```
add = a b -> a + b

add (-> 2, 4)

=> 6
```

## Calling a function

Just as with declaration, calling a function has no ceremony, it's:

```
function_name arg1 arg2 argN ...
```

For example

```
add = a b -> a + b

add 2 2
=> 4
```

You can use parenthesis to be explicit about which parameters go to which
function

```
add = a b -> a + b
sub = a b -> a - b

// Explicit
value = add 2 (sub 3 1)
dump value
=> 4

// Implicit
value = add 2 sub 3 1
dump value
=> Error: The function `sub` wants to 2 parameters, received....
```

^ That error message can be confusing because of how _lang_ tries to guess at
what you want.

In this particular case, the function `add` takes two parameters of any type
which causes _lang_ to pass the value "2" and the function "sub" as the
parameters to add.

Then the values of "3" and "1" are actually passed to `add` as "rest
parameters".

Finally, when _lang_ tries to run `add` it complains because the function `sub`
is missing the 2 parameters it declared.

The "Explicit" version is declared in such a way that tells _lang_ that `add`
should actually get the value from the result of calling `sub` with the values
"3" and "1"

_lang_ is trying to be ulta dynamic out of the gate with...

### Auto args

By default, you can actually pass anything into a function regardless of the
"arity" of your function, and _lang_ will try and make the best of it

For example

```
// You can have a function declared like this meaning that it takes no
// parameters
noop -> {}

// And call it like it expects
noop
=> Nothing

// but you can also call it like this
noop 1 2 3
=> Nothing
```

_lang_ refers to this "auto args"

Basically _lang_ will accept any _extra_ parameters passed into a
function and assign them to the <a ref="">list</a> variable `args` in your function body

```
add -> {
  stdout args
}

add 1 2 3 4
=> [1, 2, 3, 4]
```

If you want to assign another name to `args` you can do it using the variadic
syntax

```
add = ...nums -> {
  // _lang_ will assign args to "nums" in this case
  // nums = args
}
```

Of course you can increase the strictness and specifically say "no, you take
no parameters dammit" and turn off the auto\_args feature all together

```
coinfig.no.auto_args {
  add -> {}

  add 1 2
  => Error: `add` takes no parameters, given....
}
```

### Spread

You can also "spread out" a list when calling a function by sending each item in
a list to the function as a separate parameter

```
add a b c -> a + b + c

stuff = [1, 2, 3 ]

add ...stuff
=> 6
```

## Overloading

One of the core goals of _lang_ is to let you turn on the strictness as you
need it to allow you to "code jazz" while your working out an idea but then
later turn up the "type-y-ness" when/if you want

One of the ways to do this is by using the built pattern matching functionality
to define a function that different arities and return types (_lang_ calls this
"signatures")

For Example:

```
log_a_thing = f -> {
  stdout "Start"
  f
  stdout "End"
}

log_a_thing = fn:list -> {
  stdout "Run list"
  every log_a_thing fn
  stdout "Finish list"
}

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

The function `log_a_thing` has 2 signatures declared: One where anything can be
passed into it and another that accepts a list of anythings.

When _lang_ sees you define a function with multiple signatures, it will then
assume that you've covered all the signatures you want to accept so if you call
your function with a set of parameters it can't match then you'll get an error:

```
log_a_thing 1 2 3
=> Error: `log_a_thing` expects either 1 parameter....
```

Of course you can fix this by just adding this signature in your definitions:

```
log_a_thing = f -> {...}
log_a_thing = f:list -> {...}
log_a_thing -> {...}
```

Now _lang_ will let either 1 thing, a list of things, or anythings


## Functional

At it's core, _lang_ is a functional language meaning that certain features
are baked in

### Function are first-class citizens

Which means you can pass them like any other "value"

As a parameter

```
add = a b -> a + b

n -> rand 1 10

add n 4
```

Return them and declare them in other functions

```
add = a b -> a + b
sub = a b -> a - b

add2Sub3 = a -> compose (add 2) (sub 3)

add2Sub3 1
=> 0
```

On the fly as values pretty much anywhere, they're basically any other value

```
nums = [1..100]

add_inc = map (i -> + i) nums
=> [function, function, func....]

every (el, idx -> el idx) add_inc
=> [2, 4, 6, 8, ....]
```

### Auto curry / partial application

Just as how when you pass "extra" parameters to a function and _lang_ does you a
solid and presents them to you in your function, you pass a function "less"
parameters than it expected, _lang_ will automatically create a partially
applied function for you

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
you don't want to auto curry; you can disable it ad-hoc on the fly via a
<a>config block</a>

```
add = a b -> a + b

// Temporarily disable auto-curry
config.no.auto_curry {
  add2 add 2
}

=> Runtime error: add wanted 2 parameters, received only 1 on line ....
```

### Auto infix

Just about everything in _lang_ is a function, including most operators

For example the operator `+` is actually a function that has a definition
_similar_ to this

```
add = a:num, b:num -> a + b
+ = add
```

Meaning that  the following two expressions are equivalent

```
// Called like any other function
+ 1 1

// Called in "infix" position
1 + 1
```

_lang_ will automatically guess if your function is in infix position based on
the context of the parser

You can explicity tell the parser that a function is in infix position by using
the infix operator which is `<function_name>` for example

```
// Let the parser guess at positioning
1 + 1

// Tell the parser explicity that `+` is in infix position
1 <+> 1
```

#### Order of operations / precedence

Examples of auto\_infix and alternatives

```
// The following is using the `divide` and `plus` function in infix position
4 / 4 + 4
=> 5

// You could be explicit if you liked to
4 </> 4 <+> 4
=> 5

// Or you can do it without infix
+ 4 (/ 4 4)
=> 5
```

#### With examples `auto_curry`

Examples of auto\_infix and auto\_curry working hand and hand

```
x = 4 /
result = x 4
=> 1
```
