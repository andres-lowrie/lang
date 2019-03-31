---
weight: 2
title: Interfaces
---

# Interfaces

In _lang_ an "interface" is a way to ensure that a certain set of functions are
available for a given type

The syntax is 

```
binding_name = interface grammar.block:interface
```

For example, say you have some types defined like this:

```
automobile = type {
  make: string,
  model: string,
  name: (-> automobile.make + automobile.model)
}

person = type {
  first: string,
  last: string
  name: (-> person.first + person.last)
}

animal = type {
  name: string
}

alias = type {
  name: list:str
}
```

And you want to make a function that prints out the name of each of these
types, you could write

```
print = a -> "a's name is: {{a.name}}"
```

This is fine up until the you pass in something that doesn't have a "name"
function available; if that happens then you get a runtime error:

```
no_name = { foo: 'bar' }

print no_name
=> Runtime Error: `no_name` doesn't have a field or function `name`....
```

To prevent this behavior, you could define an `interface`:

```
nameable = inteface { name }

print = a:nameable -> "a's name is: {{a.name}}"
```

Now only types that have a function named "name" would be allowed as arguments
to the `print` function

## The block argument

The block that the `interface` function accepts is very expressive and allows
for a fine grained level of strictness.

The most lax version of it is one where only the function names are defined:

```
fooer = interface {
  foo,
  bar,
  baz
}
```

This is saying that a "fooer" is any type that has the functions "foo, bar, and
baz" available.

Given how the interface was defined, those function could have
any signature and do anything just as long as they exist then that's okay.

```
// If we define some functions with the correct names
foo -> 1
bar -> 'hello'
baz -> 3

// Now everything is a "fooer"
x:fooer = 1
x:fooer = 'blah'
x:fooer = ()
```

This is because "fooer" didn't define function signatures and thus the function
declared are available for any type.

If we wanted to lock that down we would define add strictness to the function
signatures:

```
fooer = interface {
  foo: str,
  bar: str,
  baz: str
}
```

Now we're saying that the "foo, bar, and baz" functions must take 1 parameter
of type string.

We can also specify the return type as well:

```
fooer = interface {
  foo: str -> str,
  bar: str -> str,
  baz: str -> str
}
```

If the functions are ones with multiple parameters, you can do that as well:


```
fooer = interface {
  foo: str -> str,
  bar: str -> str,
  baz: a b:str -> int
}
```

So if we take that last "fooer" definition then our example now breaks as
expected:

```
foo -> 1
bar -> 'hello'
baz -> 3

// Fail because no functions named "foo, bar, and baz" were found with the
// required signatures
x:fooer = 1
x:fooer = ()

// Even though a function "bar" was found with the correct signature, "bar and
// baz" are still not satisfied and therefore the assignment fails
x:fooer = 'blah'
=> Error: "blah" does not implement "fooer"
```

## Composability

Just as with `types`, you can compose interfaces to make other interfaces by
"embedding" them in the declaration

```
printable = interface { print }
nameable = interface { name }

writable = interface {
  printable,
  nameable
}

x = type {
  print: x
  name: "X is {{x.print}}"
}

printable x
=> true

nameable x
=> true

writable x
=> true
```

You can also create interfaces on the fly using:

```
writable = printable + nameable

typeof writable
=> interface
```

You can also derive interfaces by composing `types`:

```
person = type { name: string }
car = type { make: string, model: string, year_of_prod: 2019 }

nameable = interface { person, car }

dump nameable
=> interface { name: string, make: string, model: string, year_of_prod: int }
```

You can mix and match `types` and `interfaces` as well:

```
i = interface { a }
b = type { foo: "fubar" }

mixed = interface { i , b }

dump mixed
=> interface { a: anything, foo: string }
```

This works because of how _lang_ treats fields in a type.

When a `type` is defined, it will define a function that will return a
structure of a certain shape meaning that it will have certain fields tied to
it.

```
automobile = type {
  make: string,
  model: string,
  name: (-> automobile.make + automobile.model)
}
```

Here we're basically saying that the "automobile" type will have the fields
"make", "model", and "name" where:

- 'make' is of type string
- 'model' is of type string
- and 'name' is a function

But... in _lang_ everything is a function... including fields

**So what we're really saying is:**

The "automobile" type will have the fields "make", "model", and "name" where:

- 'make' is value function returing a string
- 'model' is value function returing a string
- and 'name' is a function that returns anything

So in practice, the only difference between a `type` and an `interface` is that an
interface cannot be instantiated...
