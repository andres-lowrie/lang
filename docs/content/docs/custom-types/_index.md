---
title: "Custom Types"
---

# Custom Types

Types are a core component to _lang_ and the language tries to add a lot of sugar and quality of life features to let users build off of the type system.

## Aliases

The first thing you might want to try and do is "aliasing" a type to some other name, and just like with everything else it's just a matter of "assigning" a type to a variable, a trvial example might be:

```
yes_or_no = bool
```

So now our program has a new type called `yes_or_no` which is equivalent to `bool`.

Another perhaps more useful alias could be used to write your code in another language

```
entero = int
flotante = float

// etc. etc.
```

## Declaring

_lang_ has a function named `type` that is used to define a type. You use it like any other function:
```
my_type = type <some structure>
```

### Structures / Shapes

It's common to use types in terms of structures; in _lang_ the term _"shape"_ is also used to describe these structures

For example 
```
person = type {
  first:str,
  last:str
}

// or in one line if you like
person = type { first:str, last:str }
```

You can also declare a type and assign defaults to the fields

```
person = type {
  first:str,
  last = 'Doe'
}

// Notice that the field 'last' doesn't have a type, _lang_ will infer the type,
// you could however be explicit

person = type {
  first:str,
  last:str = 'Doe'
}

// Both of these yield the same type
```

Now whenever a `person` is passed around, it will always have the field `last`
set and the value will **always** be the string 'Doe'; in other words when
assigning values to a `type`, those values are _immutable_.

### Enforcing only keys / Outlines

When roughing out some code to just get the idea on paper, it can be helpful to have a shape that only enforces the keys or _fields_ of a structure but not the values.

_lang_ calls these shapes _"outlines"_

Verbosely the way to define them is like this:
```
person = type {
  first:any,
  last:any,
  middle:any,
  address: any,
}
```

but you can omit the values if they're all going to be un-enforced
```
person = type { first, last, middle, address }
```

Note that if you're going to nest shapes, but still want to treat them as outlines, _lang_ can infer that too:

```
person = type {
  first,
  last,
  middle,
  address: [{
    number, street, city, state, zip
  }]
}
```

When types like these are declared _lang_ will ensure that the keys are met whenever it encounters this type; either at compile time (when possible) or run time when dealing with dynamic data [read more about validtion types]()

## Composing

You can build types by composing types together

```
cat = type { is_feline = true }
dog = type { is_canine = true }
catdog = type { cat, dog }

dump catdog
=> catdog { is_feline: true, is_canine: true }
```

However when embedding types, none of the types can share field names:

```
cat = type { say: 'meow' }
dog = type { say: 'bark' }
catdog = type { cat, dog }

=> Compile error: Ambiguous "say" field....
```

## Instantiation

When you declare a `type` you're actually declaring a function.

When you do

```
person = type { first:str, last:str }
```

You're declaring a function named `person` that takes an optional single
parameter of type _"structure"_.

Abstractly the singature for the type function looks something like this:

```
type = a -> f -> s:optional -> t
```

Meaning that to "instantiate" a person, you call the function just like any
other function and pass in a structure:

```
person = type { first:str, last:str }

developer = person { first: 'Jane', last: 'Doe' }

typeof developer
=> person
```

[go down the rabbit hole of TypeFunction macros]()

## Dynamic Types

_lang_ provides a way to create type restrictions during runtime using "Dynamic
Types"

These are being determined at runtime, so note that violations on these "types"
will cause the program to throw an error unlike the other types described thus
far which will cause compile time errors.

### Overview

Given that the `type` construct is just a function, you can actually leverage
that to create types on the fly.

```
// Some function that returns a structure
login ->
  username = prompt "What's your username?"
  password = prompt_secret "What's your password?"

  // Assume this returns some structure
  do_actual_login username, password
  
  
local_user -> (type getData)

=> "What's your username?"
> "the dude"

=> "What's your password?"
> *******

// The real power can come from the fact that the has
// a type defined for _this_ user specifically so
// we could define functions that only work with
// this literal user

// perhaps things like 
write_a_blog_post:local_user -> noop

// or
logout_other_sessions => (u:local_user) -> noop

// or if you suppose this is running on some shared server
// you could imagine a function somewhere that
// listens for connections, perhaps it could do
// something like
is_that_user_this_user = (a:that_user, b:local_user) -> noop

// etc. etc.
```

### Conditional Types / Guards

The _type_ function can also be used to express a condition which must evaluate to true before a value can be bound to a variable and allow the program to continue; _lang_ calls these guards

 Conditional Types are expressed via single parameter boolean functions (aka "unary" functions)

For example:

```
// In this case it is a function that always returns true
yes = (a) -> true

// This means that when "yes" is used as a type, _lang_ will let anything be
// assigned to the variable on the left hand side... because the `yes` function
// will always evaluate to true

x:yes = 1
x:yes = 'x'
x:yes = []
```

There aren't any restrictions to the function: _lang_ will run the function
passing whatever is on the right hand side of the "=" to it and check the
return value; if it's `true` lang will allow the assignment otherwise it will
return an error

For example:

```
even = (a) -> a % 2 == 0
odd = (a) -> not even

// Now we can say that "x" should only ever be a value that's even
// and "y" should only ever be a value that is odd

x:even = 3
=> Runtime error: `x` can only be assigned values that `even` returns `true` ...

y:odd = 1

z:odd = 4 / 2
=> Runtime error: `z` can only be assigned values that `odd` returns `true` ...
```

A more pratical application of this would be to combine functions together to
create a condition that can clearly express the intended value of a variable:

```
profanity_en = ... // Function that gets a list of bad words in english
profanity_es = ... // Function that gets a list of bad words in spanish

allowed_word = (w) -> not_in profanity_en + profanity_es

// Now we have a check that clearly shows what type of vlaues we want to allow
user_name:allowed_word = ...
```

In other words "conditional type" are like syntactic sugar that make it possible
to turn something like this

```
cond_fn = a ->
  if <some condition of a> is false
    error "{a} not allowed to be assigned"
  a

variable = cond_fn <potential value>
```

Into one line that clearly shows the intent

```
variable:condition = <potential value>
```

For typical conditional assignment see the section on <a>if and conditions</a>
