---
title: Interfaces
---

# Interfaces

In _lang_ an "interface" is a way to ensure that a certain set of functions are
available for a given type

For example, say you have some types defined like this across a bunch of files:

```
automobile = type {
  make: str,
  model: str,
}
name = a:automobile -> "{a.make}, {a.model}"

person = type {
  first: str,
  last: str,
}
name = a:person -> "{a.first}, {a.last}"

animal = type {
  name: str
}
name = a:animal -> a.name

aliases = type {
  name: list:str
}
name = a:aliases -> join a.name
```

And you want to make a function that prints out the name of each of these
types, you could write

```
print = a -> "a's name is: {a.name}"
```

This is fine up until the you pass in something that doesn't have a "name"
function available; if that happens then you get a runtime error:

```
no_name = { foo: 'bar' }

print no_name
=> Runtime error: `no_name` doesn't have a field or function `name`....
```

To prevent this error from happening, you could define an `interface` instead:

```
nameable = interface { name }

print = a:nameable -> "a's name is: {name a}"
```

Now only types that have a function (or field) `"name"` associated to it will
be allowed as arguments to the `print` function, if you try and run that
program now, it will fail at compile time instead of runtime

```
no_name = { foo: 'bar' }

print no_name
=> Compile error: `no_name` doesn't have a field or function `name`....
```


## The shape parameter

The `interface` function accepts a structure that takes on many shapes.

The most lax version of it is one where only the names are defined:

```
fooer = interface {
  foo,
  bar,
  baz
}
```


This is saying that a "fooer" is any type that has "foo, bar, and baz"
function or field available.

So we could define a struct with those fields and the that _thing_ would implement the interface

```
thing = {
  foo: "foo",
  bar: 1,
  baz: false
}

fooer thing
=> true
```

If we wanted to be more strict about types we can do so

```
fooer = interface {
  foo: str,
  bar: str,
  baz: str
}
```

Now we're saying that the "foo, bar, and baz" must be fields of a certain type and we're saying that those fields must be of type string.

If wanted to instead say that they should be functions, we would declare them as such:

```
fooer = interface {
  foo: fn,
  bar: fn,
  baz: fn
}
```

and we could add types to those as well:
```
fooer = interface {
  foo: fn:str,
  bar: fn:str,
  baz: fn:str
}
```

We can also specify what _signatures_ the function should have:

```
fooer = interface {
  foo: -> any,
  bar: (:int) -> int,
  // can also use the `tio` function
  baz: :str:,
}
```

You can specify functions with multiple parameters as well. You would just omit the parameter name:

```
fooer = interface {
  foo: str -> int,
  bar: int -> str,
  baz: (str, str) -> int
}
```

## Composability

Just as with `types`, you can compose interfaces to make other interfaces

```
printable = interface { print }
nameable = interface { name }

writable = interface {
  printable,
  nameable
}

x = {
  name: "equis",
  print: () -> "X is {x.name}"
}

printable x
=> true

nameable x
=> true

writable x
=> true
```

You can also create interfaces on the fly:

```
writable = printable + nameable

typeof writable
=> interface
```

## Deriving

### fields from types

You can also _"derive"_ interfaces by composing `types`.

When the `interface` function is called with types, it will look for all the fields associated of the passed in types and define an interface with all those types

```
person = type { first:str, last:str }
name = p:person -> "{p.first} {p.last}"


car = type { make: str, model: str, year_of_prod: 2019 }
pretty_name = c:car -> "{c.year_of_prod} {c.make} {c.model}"


nameable = interface { person, car }

dump nameable
=> interface { fist:str, last:str, make:str, mode:str, year_of_prod: int...
```

However, if any of the types share a field name with conflicting signatures then you'll get an error

```
person = type { first:str, last:str }

birthdays = type { first:date, last:maybe:date }

grim = interface { person, birthdays }

=> Compile error: `person` and `birthdays` have different signatures for...
```

### pruning fields

`interface` accepts a predicate function as a parameter that allows you to limit which fields are derived:
```
person = type { first:str, last:str }
name = p:person -> "{p.first} {p.last}"

car = type { make: str, model: str, year_of_prod: 2019 }
pretty_name = c:car -> "{c.year_of_prod} {c.make} {c.model}"

// Here we can say we only want the functions that are available
nameable = interface { person, car }, (typeof fn)

dump nameable
=> interface { name: person -> str, pretty_name:car -> str }

```


### from mixed

You can also mix and match `types` and `interfaces` as well, with all same rules as outlined above:

```
i = interface { a }

report = type {
  foo: "fubar"
}
sitrep = (report) -> report.foo


mixed = interface { i , report }

dump mixed
=> interface { a: anything, sitrep:report -> str }
```

## Immutable

Interfaces are immutable by default and are only open to changes during parse time, meaning that you **can't** change the types an interface points to after you've declared an interface, regardless if those structures are mutable or not; doing so causes an error

For example the following code will break
```
animal = mut { breed: str }

i = interface { animal }

animal.size = 'small'
=> Compile error: `animal` was referenced by `my_interface` by on ...
```

If you want the interface to be "updated" whenever it's underlyin structures change, then you need to make explicitly define:
```
animal = mut { breed: str }

i = mut interface, { animal }

animal.size = 'small'
```

They're caveats, compile time, and run time quirks to how strict the interface can be if declared to be `mutable`, [go down the rabbit hole]()
