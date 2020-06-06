---
title: "Custom Types"
---

# Custom Types

## Declaring

Sometimes you want to define the type of things that are in a structure, for
that you can use the `type constructor` syntax is:

```
binding_name = type {
  field_name:[<type>]
  ...
}
```

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

On the other hand if you just want a type to use a shape, ie: the
actual _type_ of the fields doesn't matter, then you can use the `anything` type:

```
person = type {
  first:anything,
  last:anything
}
```

The type constructor will default to the anything type if none is specified, so
if you just wanna make shapes you can use the shorthand

```
person = type { first, last }
```


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

=> Compile Error: Ambiguous "say" field....
```

## Instantiation

When you declare a `type` you're actually declaring a function.

When you do

```
person = type { first:str, last:str }
```

You're declaring a function named `person` that takes an optional single
parameter of type _"structure"_.

The singature for the type function looks this:

```
type -> a:struct -> b -> a
```


Meaning that to "instantiate" a person, you call the function just like any
other function and pass in a structure:

```
person = type { first:str, last:str }

developer = person { first: 'Jane', last: 'Doe' }

dump type_of developer
=> person
```

The single parameter is always optional for a type so you could also do the
following to achieve the same thing:

```
developer = call person
developer.first = 'Jane'
developer.last = 'Doe'
```

## Dynamic Types

Given that the `type` construct is just a function, you can actually leverage
that to create types on the fly.

```
getData ->
  first = prompt "What's your name?"
  last = prompt "What's your last name?"
  {first, last}

// Here we say that want to define the structure `getData` returns as a custom
// type
user = type getData

// Then we can run out program
call user
dump user

=> "What's your name?"
> Bobby

=> "What's your last name?"
> Sue

=> user { first:'Bobby', last: 'Sue' }
```

## Conditional Types

The type system in _lang_ is not only optional but it also strives to be
conditional; the idea is to allow the user to be able to express a condition
which must evaluate to true before a value can be bound to a variable.

The conditions are expressed via single parameter ("unary") boolean functions

For example:

```
// We declare a unary function
//
// In this case it is a function that always returns true
yes = a -> true

// This means that when "yes" is used as a type, _lang_ will let anything be
// assigned to the variable on the left hand side... because the `yes` function
// will always evaluate to true

x:yes = 1
x:yes = 'x'
x:yes = []
```

It doesn't matter what the function does _lang_ will run the function passing
whatever is on the right hand side of the "=" and check the return value; if
it's `true` lang will allow the assignment

For example:

```
even = a -> a % 2 == 0
odd = a -> not even

// Now we can say that "x" should only ever be a value that's even
// and "y" should only ever be a value that is odd

x:even = 3
=> Runtime Error: `x` can only be assigned values that `even` returns `true` ...

y:odd = 1

z:odd = 4 / 2
=> Runtime Error: `z` can only be assigned values that `odd` returns `true` ...
```

A more pratical application of this would be to combine functions together to
create a condition that can clearly express the intended value of variable:

```
profanity_en = ... // Function that gets a list of bad words in english
profanity_es = ... // Function that gets a list of bad words in spanish

allowed_word = w -> not in profanity_en + profanity_es

x:allowed_word = ...
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

Into one line that shows more clearly shows the intent

```
variable:condition = <potential value>
```

For typical conditional assignment see the section on <a>if and conditions</a>
