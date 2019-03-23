---
weight: 3
title: "Control Flow"
---

# Control Flow

_lang_ is a functional, expression based language so there really isn't typical
"control flow" per say, but the syntax allows you to mimic it

## if

_lang_ only ships with 1 conditional function, and that's the
quintessential `if`

It's defined like

```
if = grammar.eval true grammar.runnable grammar.optional grammar.runnable
```

but before we dive into that let's look at from a syntax perspective

### Do something if something is true

The most simple thing to check is a boolean value directly

```
if true {
  // do something
}
```

You can't use the `=` operator because that's used by the compiler but you can
use the `is` function to compare things

```
a = 'a'

if a is 'a' {
  stdout "it's an a"
}
```

You can also do "truthy" and "falsey" style 

```
yes = 'has a "none zero" value'

if yes { 
  stdout "Yaaaass!"
}

yes = ''

if !yes {
  stdout "yes is falsey"
}
```

### Do something if multiple things are true

You can compare multiple things ... <a
href="https://img.memecdn.com/when-im-waiting-for-someone-to-react-to-my-joke_gp_1906935.webp">
if </a> .... you like 

```
yes = true
no = false

// both
if yes and no {
  stdout "won't make it here"
}

// either
if yes or no {
  stdout "Will see these"
}
```

You can add as many "conditions" as you like

```
a = 1
b = 2
c = 3

if a and b and c { 
  stdout 'all 3 are truthy'
}
```

## What about else

If we look back at how the `if` function is declared, you'll see that the second
part of the function is an "optional" runnable. That runnable is what _lang_
will run if the condition is not true

In other words

```
if false {} { // this one will run}
```

So if you wanted to do something when the condition isn't true out of the box
you could do

```
a = rand bool

if a {
  // do if a is true
} {
  // do something if a is false 
}
```

But that looks weird, with the bracket syntax. It looks natrual without it

```
a = rand bool

do_it -> {}
dont_do_it -> {}


if a do_it dont_do_it
```

If that still looks weird, we can always just create an `else` function and use
that instead

```
else = block:grammar.runnable -> block

a = rand bool

if a {
  // do if a is true
} else { 
  // do if a is not true
}
```

That works because `else` is a function that takes a runnable and returns it so
basially it works like a passthrough

<a>more info on hacking</a>

## What about `else if`

We can build on the concepts above and instead define `else` like this

```
else -> if (args.first is_a 'grammar.runnable') args.first (grammar.until !'else')

a = rand bool

if false {
  stdout "never do this"
} else if a {
  stdout "do this if a is true"
} else {
  stdout "do this if a is false"
}
```

This works because `else` is leveraging the parser here and will slurp up everything
passed to it until it reaches an `else` statement

## Well, I like `elif`

If you're into python's elif ... I mean you can define your functions to mimic
that style instead

```
elif = if
else = block:grammar.runnable -> block

a = rand bool

if false {
  stdout "never do this"
} elif a {
  stdout "do this if a is true"
} else {
  stdout "do this if a is false"
}
```

_lang_ let's you define functions that make sense for you

<a>More info on hacking</a>
