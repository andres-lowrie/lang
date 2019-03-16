---
weight: 5
---

# Functions

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

Like everything else, you can disable auto curry via the [config]() if you like
... but you don't want to do that. :wink:


