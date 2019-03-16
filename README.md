language
========

## It should

- Be configurable and extensible at **every** point
- Be declarative over imperative (functional)
- Be as strict as you make it but guess everywhere else
- Be good at guessing
- Have the compiler do as much work as possible
- Allow testing to be a first class citizen while still letting the code say what
  it wants to say (test should bend to code not otherway around)
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

// You always have the final say though
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

## Complex Types

### First class JSON

The structures in _lang_ are basically JSON

```
data = {
  foo: 'bar',
  num: 123,
}
```

This should create a structure called `data` with fields `foo` of type string and
`bar` of type integer

You should not be limited to primitive types, you can also add functions as field

```
data = {
  foo: 'bar',
  num: 123,
  f: a b -> a + b,
}

```

Here `data` now also has a function associated to it, `f`

You can also call the function when declaring a structure

```
data = {
  foo: 'bar',
  num: 123,
  pi: (-> 3.14)()
}
```

This will add `pi` with the float value of `3.14` to `data` structure

### Shorthands

You can pass in variables and the structure will assign key:value pairs to the
structure using the variable name and value

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

### Strictness and types

You can also declare structures that must confirm to a certain shape using the
`type` function

```
data = type { 
  num: int,
  foo: string,
  pi: float
}
```

This will create a function called `data` that can be used to create structures
that must adhere to specified shape 

```

my_data = data()
=> data {num: 0, foo: '', pi: 0.0}

```

You can also mix and match and the let the compiler fill in the wholes

```
data = { 
  num: int,
  foo: string,
  pi()
}
=> data {num: 456, foo: 'bar', pi: 3.14}
```


## Variables

Very little ceremony

```
foo = 1
bar = 'something'
```

Basically it's 

```
something = // anything
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

You can also create a function that takes parameters just as easy.

```
add = a b -> {
  a + b
}
```

The return is implicit as in the last expression will be returned, but you can
also use the `return` keyword if you like

```
add = a b -> {
  return a + b
}
```

Also note that the body is completly optional:

```
add = a b -> a + b
```


### Parameters

You can declare functions that don't take any parameters as well:

```
add = -> {
  // do something
}

// But that equal sign looks weird, so you can leave it out
add -> {}
```

You can also declare function that take both positional and variadic parameters

```
add = a b ...others -> {
  // do something
}
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

By default, _lang_ will be hyper dynamic. If your function is expressed without
explicit parameters then it will accept variadic parameters assigned to the
[list]() variable `args` in your function body

```
add -> {
  stdout args
}

add 1 2 3 4
=> [1, 2, 3, 4]
```

If you want to assign another name to `args` you can do either via [config]() or
when you declare your function using the variadic syntax

```
add ...nums -> {
  // The compiler will assign args to nums in this case
  // nums = args
}
```

Of course you can increase the strictness and specifically say "no, you take
no parameters dammit"

```
add = () -> {}
```

You can also spread out a list when calling a function

```
add a b c -> a + b + c

stuff = [1, 2, 3 ]

add ...stuff
=> 6
```


### Functional

At it's core, _lang_ is a functional language meaning


#### That by default all functions are curried

```
add = a b -> a + b

add2 = add 2

// Since add expected 2 parameters but got 1, it will return a function instead
// of expression of it's body

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
config.no_auto_curry {
  add2 add 2
}

=> Runtime error: add wanted 2 parameters, received only 1 on line ....
```

Like everything else, you can disable auto curry via the [config]() if you like
... but you don't want to do that. :wink:

#### Functions are first-class citizens

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

// Here we're using the composer shorthand `.`
add2Sub3 = a -> sub 3 . add 2

add2Sub3 4
=> -3
```

On the fly

```
add = a b -> a + b

add (rand 1 10) 4
```

By default _lang_ will call your function when it appears anywhere (except when
it's being declared, or passed as parameters)

You can disable this and have functions be called explicitly via the [config](),
or a [scope block]().

If you so choose this route, you can call functions with the parenthesis `()`:

```

config.explicit_call {
  add = a b -> a + b

  n -> rand(1, 10)

  add n() 4
}
```
