---
title: "Control Flow"
---

# Control Flow

_lang_ only ships with 1 typical control flow structure, and that's the
quintessential `if`.

## if

However it's not a statement or special syntax it's just a function, it's
defined like this

```
if = (true, branch:expression, _) -> branch
if = (false, _, branch:exression) -> branch
```

Basically it uses pattern matching to call a different function (or
"branch") depending on the value of some condition; this pattern matching
approach allows _lang_ to use `if` like any other function which comes with
some nice benefits.

### Do something if something is true

The most simple case

```
if true
  // do something
```

If you want to compare something, then you can use the equality operator
(ummm actually technically it's a function \*pushes up glasses)

```
something = 1

if something == 1
  stdout "something is 1"
```

### Multiple Conditions

You can use the `and` operator or the `or` operator


```
yes = true
no = false

// both
if yes and no
  stdout "won't make it here"

// either
if yes or no
  stdout "Will see these"
```

You can add as many "conditions" as you like and group them using parenthesis

```
a = 1
b = 2
c = 3

if a and b and c
  stdout 'all 3 are truthy'

if (a and b) or c
  stdout 'all 3 are truthy'

if a or (b or c)
  stdout 'all 3 are truthy'
```

However since all these "operators" are just functions returning things, you
can also store conditions and create more expressive code

```
a = 1
b = 2
c = 3

cond1 = a == 1 and b /= 4
cond2 = cond1 and c == 3

if cond1 and cond2
  stdout 'all 3 are truthy'
```

## What about else

`if` is a function that has a signature that will ignore the first expression passed to it and run the second when the first argument is false

```
if false <ignored> <called_instead>
```

Which means that there is no `else` and the out of the box "if/else" looks like this:


```
a = rand bool

if a
  stdout 'a is true'
  stdout 'a is false'
```

This is by design, _lang_ doesn't ship with an `else` on purpose, it becomes
too easy to create nested code that is really hard to read, instead it wants
you to pass in a function

Here's the type of thing this system tries to avoid

```
if condition
  if another_condition
    run_when_true
  else
    run_when_false
else
  if yet_another_condition
    something_else
  else
    run_this_thing
```

Reading code that looks like this sucks, _lang_ wants you to keep a left
aligned clean "line of sight".

Given that:
- conditions are just functions
- functions can be stored into variables
- `if` is just any other function
- and functional constructs like partial application come for free

it's pretty easy to take something that would've looked like the above code and make it something much easier to read

```
// Move the nested stuff up and out
nested_happy = if another_condition, run_when_true, run_when_false
nested_sad = if yet_another_condition, something_else, run_this_thing

if condition
  nested_happy
  nested_sad
  
  
// Partial application can also be used to create much more complex
// flows that are easily readable
nested_happy = if _, run_when_true, run_when_false
nested_sad = if _, something_else, run_this_thing

run =
  if _
    nested_happy another_condition
    nested_sad yet_another_condition

// and now we can pass the logic to run something given some condition
// or use it to build even more complex, deeply nested code.
//
// The idea with _lang_ is to push that complexity to the parser and
// run time instead of our code
run condition
```


## Matching

Sometimes you want to run a different function given a bunch of different conditions, 
cases where a `switch` statement could be used in other languages.

For those cases you can use the `match` function. In very simple terms it has a function that looks like something like this

```
match = (a:any, b:struct) -> // implementation
```

The thing to note in this function is the `b` parameter is not a simple structure, it's actually a different type that expects the keys of the structure to be `unary` predicate functions. The implementation of `match` will then pass `a` to each of those functions and will run the corresponding function in the pair for the first predicate that returns true.

Sounds complicated but it's actually simplier to just look at the usage.

> Note that the [typeof]() function has a very robust signature, this is example is using [auto currying]() to create a predicate function on the fly

```
match "match is just a function" {
  (typeof int): stdout "its an integer",
  (typeof str): stdout "its a str"
}
```


## Sugary goodness

Given the "line of sight" initiative, _lang_ ships with a bunch of functions to make reading and conditions easier

### unless

The inverse of `if`, a nice shortcut to do say "do this unless this is true"

```
cond = rand bool

unless cond
  stdout "cond evaluated to false"
```

### equality functions

All of these mean the same thing, "both are equal"

```
a == b
a is b
a equal b
```

### non-equality functions

All of these mean the same thing, "both are not equal"

```
a /= b
a <> b
a isnt b
a not_equal b
```

### negation

the function `not` is similar to `!` in other languages

```
a = true

if not a
  stdout "you should use `unless` instead"
```
