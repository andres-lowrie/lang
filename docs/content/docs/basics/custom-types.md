# Custom Types

## Declaring

Sometimes you want to define the type of things that are in a structure, for
that you can use the `type constructor` syntax is:

```
binding_name = type {
  field_name: type
  ...
}
```

For example 
```
person = type {
  first: str,
  last: str
}

// or in one line if you like
person = type { first: str, last: str }
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
set and the value will **always** be the string 'Doe; in other words when
assigning values to a `type`, those values are immutable.

On the other hand if you just want to a type to use a shape, ie: the
actual _type_ of the fields doesn't matter, then you can use the `anything` type:

```
person = type { 
  first: anything,
  last: anything
}
```

The type constructor will default to the anything type if none is specified, so
if you just wanna make shapes you can use the shorthand

```
person = type { first, last }
```


## Embedding

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

=> Error: Ambiguous "say" field....
```

## Instantiating

When you declare a `type` you're actually declaring a function.

When you do

```
person = type { first: str, last:str }
```

You're declaring a function named `person` that takes an optional single
paramter of type _"structure"_

Meaning that to "instantiate" a person, you call the function just like any
other function and pass in a structure to it

```
developer = person { first: 'Jane', last: 'Doe' }
```

The single paramter is always optional for a type so you could also do the
following to acheive the same thing:

```
developer = person
developer.first = 'Jane'
developer.last = 'Doe'
```

## Type Function

You can leverage the functional nature of _lang_ to create type functions on the fly as
well

```
getData ->
  first = prompt "What's your name?"
  last = prompt "What's your last name?"
  {first, last}

user = type getData
stdout user

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

The conditions are expressed via single paramter ("unary") boolean functions

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
=> Error: `x` can only be assigned values that `even` returns `true` for....

y:odd = 1

z:odd = 4 / 2
=> Error: `z` can only be assigned values that `odd` returns `true` for....
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
  if <some condition of (a)> is not true
    error "${a} not allowed to be assigned"
  a

variable = cond_fn <potential value>
```

Into one line with a much clearer syntax

```
variable:condition = <potential value>
```

For typical conditional assignment see the section on <a>if and conditions</a>

## Mutability

Structures are basically just `types` but you're declaring and instantiating
them at the same time. 

So when you do 

```
person = {
  name: 'Jane',
  last: 'Doe'
}
```

You're actually doing this

```
person = mut type {
  name: string,
  last: string,
} { name: 'Jane', last: 'Doe'}

=> person { name: 'Jane', last: 'Doe' }
```

The difference between a structure and a type is the fact that the
structure function will return a type that is _mutable_ and the type function
returns one that isn't

Which is what that `mut` function is doing there in last example

## Ad-hoc Immutability

```
// At this point we don't know what we want to lock down
// so we'll leave it open
data = { 
  truth: 4
}

propaganda = prompt "Share truth? 2 + 2 = " + data.truth

=> Share truth? 2 + 2 = 4

> 5

data.truth = propaganda

// Now we want to lock it down, so we can easily do that
// from this point on in the program
data = type data

// thus we can modify the type anymore
data.truth = 4

=> Error: The type `data` shouldn't have its field `truth` ....
```

## Self referencing

Things get real spicy when you want to access the thing you're declaring while
you're declaring it, so _lang_ should have support for that

```
data = type { 
  first: string,
  last: string,
  name: (-> data.first + data.last)
}

// Later

d = data { 'Jane', 'Doe'}
d.name

=> 'Jane Doe'
```

## Embedding

You can also embed structures within structures and keep references to the
original structure

```
gun = { damage: 'high' }
knife = { damage: 'med' }
fist = { damage: 'low }

weapons = { main: gun, secondary: knife, aux: fist }

// Structures and are mutable by default

train = a -> a.damage = 'high'

train fist

dump weapons
=> weapons {
=>   main: gun {
=>     damage: 'high'
=>   },
=>   secondary: knife {
=>     damage: 'med'
=>   },
=>   aux: fist {
=>     damage: 'high'
=>   },
=> }
```

If you don't want references you can `copy` structures instead

```
x = { foo: 'bar' }

y = { x: copy x }
y.x = 'not bar'

stdout x y
=> x { foo: 'bar' }
=> y { foo: 'not bar' }
```

You can invert the default behavior natrually with 

```
config.struct\_embed\_use\_copy {
  // ...
}

// Or

config.no.struct\_embed\_use\_copy {
  // ...
}
```

