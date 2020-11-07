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
name = a:aliases -> join a.name " "
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
=> Runtime Error: `no_name` doesn't have a field or function `name`....
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
=> Compile Error: `no_name` doesn't have a function `name`....
```

## The shape parameter

The `interface` function takes a structure as its only parameter that takes on
many shapes.

The most lax version of it is one where only the names are defined:

```
fooer = interface {
  foo,
  bar,
  baz
}
```


This is saying that a "fooer" is any type that has "foo, bar, and baz"
function/field available.

So we could define a struct with those fields:

```
thing = {
  foo: "foo",
  bar: 1,
  baz: false
}

foo = a:thing -> a.foo
bar = a:thing -> a.bar
baz = a:thing -> a.baz

fooer thing
=> true
```

If we wanted to define a type for the function of field we would define a type
as the value of the keys we want:

```
fooer = interface {
  foo: str,
  bar: str,
  baz: str
}
```

Now we're saying that the "foo, bar, and baz" functions must take 1 parameter
of type string when they're functions or in the case that the implementation is
a field; it must be of type string.

If we want to specifically say that we want functions and not fields, We can
also specify the return type function as values:

```
fooer = interface {
  foo: str -> str,
  bar: str -> str,
  baz: str -> str
}
```

You can specify functions with multiple parameters as well:

```
fooer = interface {
  foo: str -> str,
  bar: str -> str,
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
  print: (-> "X is {x.name}")
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

type_of writable
=> interface
```

## Deriving

### from types

You can also _"derive"_ interfaces by composing `types`. When the `interface`
function is called with types, it will look for all the functions associated
with the types and define an interface specifying said types

```
person = type { first:str, last:str }
name = p:person -> "{p.first} {p.last}"


car = type { make: str, model: str, year_of_prod: 2019 }
pretty_name = c:car -> "{c.year_of_prod} {c.make} {c.model}"


nameable = interface { person, car }

dump nameable
=> interface { name:str, pretty_name:str }
```

However, if any of the types have a function associated to them that collide
with any of the types, you'll get an Error; collision here means signature
collision

```
person = type { first:str, last:str }
name = p:person -> "{p.first} {p.last}"


dog = type { nick:str }
name -> "{d.nick}"

nameable = interface { person, dog }

=> Compile Error: `person.name` takes 1 parameter, `dog.name` takes no param...
```

### from mixed

You can mix and match `types` and `interfaces` as well:

```
i = interface { a }

b = type { foo: "fubar" }
sitrep = b:b -> b.foo


mixed = interface { i , b }

dump mixed
=> interface { a: anything, sitrep: str }
```
