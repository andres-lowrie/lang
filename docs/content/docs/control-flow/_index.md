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

Basically it uses the pattern matching to call a different function (or
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

### Do something if multiple things are true

You can compare multiple things ...
[if]("https://img.memecdn.com/when-im-waiting-for-someone-to-react-to-my-joke_gp_1906935.webp")
... you like using the the `and` funciton as well as the `or` function

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

You can add as many "conditions" as you like

```
a = 1
b = 2
c = 3

if a and b and c
  stdout 'all 3 are truthy'
```

However since all these "operators" are just functions returning things, you
can also store conditions and create more expressive code

```
a = 1
b = 2
c = 3

cond1 = a == 1 and b != 4
cond2 = cond1 and c == 3

if cond1 and cond2
  stdout 'all 3 are truthy'
```

## What about else

`if` has a signature that will ignore the first expression passed to it and run
the second when the first argument is false

```
if false <expression1> <expression2>
```

Given that there aren't any braces when calling functions, the 'out of the box'
"if/else" requires the use of `do` block to denote expressions


```
a = rand bool

if a
  do stdout 'a is true'
  do stdout 'a is false'
```

This is by design, _lang_ doesn't ship with an `else` on purpose, it becomes
too easy to create nested code that is really hard to read, instead it wants
you to pass in an expression

```
a = rand bool

do_it -> stdout "doing it"
dont_do_it -> stdout "not doing it"

if a
  do do_it
  do dont_do_it
```

Here's the type of thing this system tries to avoid

```
if condition
  if another_condition
    do_something
  else
    another_thing
else
  if yet_another_condition
    blah
  else
    more_nonsense
```

Reading code that looks like this sucks, _lang_ wants you to keep a left
aligned clean "line of sight". Given that conditions can be stored into
variables, `if` is just any other function, and functional constructs like
partial application are availalbe; it's pretty easy to take something that
would look like the above code and make it something much easier to read

```
nested_happy = if _ do_something another_thing
nested_sad = if _ blah more_nonsense

if condition
  do nested_happy another_thing
  do nested_sad yet_another_condition
```

## Matching

Sometimes you want to run a different function given a bunch of different conditions, 
cases where a `switch` statement could be used in other languages.

For those cases you can use the `match` function

```
match = (a:any, b:stuct) -> any
```

The thing to note in this function is the `b` parameter which is a structure
where each key is a ("predicate function", and an expression).

The functions are actually partially applied functions such that, `match` will
pass `a` to them, if they evaluate to `true`, then the expression for the
corresponding predicate function is executed

```
match "match is just a function" {
  (type_of int): stdout "its an integer",
  (type_of str): stdout "its a str"
}
```

## Sugary goodness

Given the "line of sight" initiative, _lang_ ships with a bunch of functions to
make reading conditional easier

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
a == b  // both are equal
a is b  // both are equal
```

### non-equality functions

All of these mean the same thing, "both are not equal"

```
a != b    // both are not equal
a isnt b  // both are not equal
```

### negation

the function `not` is similar to `!` in other languages

```
a = true

if not a
  stdout "you should use `unless` instead"
```
