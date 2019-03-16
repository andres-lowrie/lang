---
weight: 4
---

# Types

## Declaring

Sometimes you want to define the type of things that are in a structure, for
that you can use the `type` function the syntax is:

```
struct = type {
  field_name: type
  ...
}
```

For example

```
person = type {
  first: string
}

// or in one line if you like
person = type { first: string }
```

You can also declare a type and assign defaults to the fields

```
person = type {
  first:string = 'Jane'
}
```

Now whenever a `person` is passed around, it will always have a field `first`
and the value will always be 'Jane'

In other words when assigning values to a `type`, those values are immutable.

### Instantiating

When you declare a `type` you're actually declaring a function.

When you do

```
person = type { first: string }
```

You're declaring a function named `person` that takes a [type block]() as its only
parameter. Meaning that to "instantiate" a person, you call the function just
like any other function.

```
// non strict
developer = person { 'Jane' }

// or strict
developer = person { first:'Jane' }
```

When using the non-strict syntax the order of the parameters will be assigned in
the same order the fields of the structure where declared in other words

```
abc = type { a:int, b:int, c:int }
easy_as = abc 1 2 3

=> abc { a:1, b:2, c:3 }
```

#### Functions everywhere (macros?)

You can leverage the functional nature of _lang_ to create types on the fly as
well

```
getData -> {
  first = prompt "What's your name?"
  last = prompt "What's your last name?"
  {first, last}
}

user = type getData
stdout user

// => "What's your name?"
// > Bobby
//
// => "What's your last name?"
// > Sue
//
// => user { first:'Bobby', last: 'Sue' }
```

#### Mutability

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

// => person { name: 'Jane', last: 'Doe' }
```

The difference between a structure and a type is the fact that the
structure function will return a type that is _mutable_ and the type function
returns one that isn't

Which is what that `mut` function is doing there in last example

#### ... and that let's you control strictness at the time you want to control it

```
// At this point we don't know what we want to lock down
// so we'll leave it open
data = { 
  truth: 4
}

propaganda = prompt "Share truth? 2 + 2 = " + data.truth

// => Share truth? 2 + 2 = 4
//
// > 5

data.truth = propaganda

// Now we want to lock it down, so we can easily do that
// from this point on in the program
data = type data

// thus we can modify the type anymore
data.truth = 4

=> Compile error: The type `data` shouldn't have its field `truth` ....
```

### Self referencing

Sometimes you want to access the thing you're declaring while you're declaring
it, so _lang_ should have support for that

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
