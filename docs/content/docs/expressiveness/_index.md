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

However the dot `.` token is used to allow users to "bind" values to other values
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
person.cheap_weapons -> filter (w -> starts_with 'm') person.weapons
person.pick_weapon = w -> find w person.weapons
```

The dot operator will let you create structures on the fly as well by omitting
the braces

```
// Here _lang_ interpreter will create a structure for the "spy" variable
// automatically based on the dot usage
spy.name = 'James Bond'
spy.email = '007@mi6.gov.uk'
=> spy { name: 'James Bound', email: '007@mi6.gov.uk' }
```

## Pipe

The pipe operator `|` can be used to pass the output of one function as input to
another function.

For example

```
phrase = "hello world, from the pipes"

x = phrase | rm ',' | strings.split_words | strings.uppercase
=> ["HELLO", "WORLD", "FROM", "THE", "PIPES"]
```

You can use newlines to keep things readble

```
str = "hello world, from the pipes"

x = str
  | rm ','
  | strings.split_words
  | strings.uppercase
=> ["HELLO", "WORLD", "FROM", "THE", "PIPES"]
```
