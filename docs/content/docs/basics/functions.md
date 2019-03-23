---
weight: 5
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
really need is the _thin arrow_ and an assignment operator

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
add = () -> {}

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

By default, the compiler will create a [body block]() around anything on the
right hand side of the thin arrow if you don't specify one.

The only time you're required to specify one (body block) is when you want to be
explicit about the return of a function

## Lambdas

You can also create an anonymous function (as in a function that is not
assigned to a variable), you just have to surround your function with parenthesis

The syntax is:

```
(params -> return body)
```

where `params` and `return` are optional.

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
// => 6
```

You can also have lambdas with multiple retuns

```
add = a b -> a + b

add (-> 2, 4)

// => 6
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

### Parameter magic

You can actually pass anything into a function regardless of the "arity" of your
function, _lang_ will make the best of it

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

_lang_ calls this "auto rest"

Basically _lang_ will accept any _extra_ parameters passed into a
function and assign them to the <a ref="">list</a> variable `args` in your function body

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
add = ...nums -> {
  // The compiler will assign args to nums in this case
  // nums = args
}
```

Of course you can increase the strictness and specifically say "no, you take
no parameters dammit"

```
coinfig.no.auto_rest {
  add = () -> {}
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

## Functional

At it's core, _lang_ is a functional language meaning that certain features
are baked in

### Function are first-class citizens

Pass them anywhere

As a variable

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

On the fly...

```
add = a b -> a + b

add (rand 1 10) 4
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
you don't want to auto curry; you can disable it ad-hoc on the fly via a [scope block]()

```
add = a b -> a + b

// Temporarily disable auto-curry
config.no.auto_curry {
  add2 add 2
}

=> Runtime error: add wanted 2 parameters, received only 1 on line ....
```

Like everything else, you can disable auto curry via the [config]() if you like
... but you don't want to do that. :wink:

### Auto infix

Just about everything in _lang_ is a function, including most operators

For example the operator `+` is actually defined like this

```
add = a:num, b:num -> a + b
+ = add
```

Meaning that `+` is actually a function and thus the following two expressions
are equivalent

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
// => 5

// You could be explicit if you liked to
4 </> 4 <+> 4
// => 5

// Or you can do it without infix
+ 4 (/ 4 4)
// => 5
```

#### With examples `auto_curry`

Examples of auto\_infix and auto\_curry working hand and hand

```
x = 4 /
result = x 4
// => 1
```
