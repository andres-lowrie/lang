---
title: Expressiveness
---

# Expressiveness

Being able to "just declare and express" things is a staple of _lang_

## Assign if empty

```
x = 'set user'

a = x or 'default'
```

## Assign from condition

```
x = 0 

a = if x true false
=> false
```


## The Magical Dot

_lang_ doesn't have a concept of objects, it only knows types and functions.

However the dot `.` token is used to allow users to "bind" types to other types
and allow for some spicy expressions

### Structures

The dot operator can be used to access fields of a structure or custom type

```
person = { first: 'Tony', last: 'Stark' }

person.first
=> Tony

person.last
=> Stark
```

You can also add fields with the dot operator and have the assignment work the
same way it does in any other place

```
person = { first: 'Tony', last: 'Stark' }

person.occupation = 'Superhero'
person.alias = 'Iron Man'
person.weapons = ['missles', 'powerbeam', 'lazers', 'machine guns']
person.cheap_weapons -> filter (w -> startsWidth 'm') person.weapons
person.pick_weapon = w -> find w person.weapons
```

The dot operator will let you create structures on the fly as well by omitting
the braces

```
person.name = 'James Bond'
person.email = '007@mi6.gov.uk'
=> person { name: 'James Bound', email: '007@mi6.gov.uk' }
```

### Types
Things start to get spicy when types and dots are in play

When the dot operator is used on a type where the right hand side of the
operator is an unknown field, then _lang_ will look for functions that can
operate on the type on the left hand side of the dot

For example

```
find = f xs:list -> {
  head, tail... = xs
  if (f head) head (find f tail)
}

[2,3,4].find(i -> i is 2)
=> 2
```

If you want to be more strict you can of course do that

```
find = f xs:list:int -> {
  head, tail... = xs
  if (f head) head (find f tail)
}

[1, "foo", false].find (i -> i % 2)
=> Runtime error: Couldn't find a function named `find` that operates on ....
```

## Pipe

The pipe operator `|` can be used to pass the output of one function as input to
another function.

For example 

```
str = "hello world, from the pipes"

x = str | rm ',' | strings.split_words | strings.uppercase
=> ["HELLO", "WORLD", "FROM", "THE", "PIPES"]
```

