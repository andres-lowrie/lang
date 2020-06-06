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
person.cheap_weapons -> filter (w -> starts_with 'm') person.weapons
person.pick_weapon = w -> find w person.weapons
```

The dot operator will let you create structures on the fly as well by omitting
the braces

```
spy.name = 'James Bond'
spy.email = '007@mi6.gov.uk'
=> spy { name: 'James Bound', email: '007@mi6.gov.uk' }
```

### Types
Things start to get spicy when types and dots are in play

When the dot operator is used on a type what _lang_ is doing is looking for a
function that matches the name on the right hand side of the dot that can
operate on the type on the left hand side of the dot

```
// Define a function called `find` that can operate on on any list
find = f xs:list ->
  head, ...tail = xs
  if (f head) head (find f tail)

// Here we're saying call `find` with list `[1, 2, 3]`
[2,3,4].find (i -> i is 2)
=> 2

// Which is equivalent to saying
find (i -> i is 2) [2, 3, 4]
```

What this is doing is basically reversing the parameter order and calling the
function to the right hand side of the dot.

This type of expression isn't limited to list of collection style types but
works on all types:

```
uppercase = s:str -> body

// Calling it like this
"hello".uppercase

// Is doing this behind the scenes in place
uppercase "hello"


// You're not limited to 1 parameter either
foo = a b c:str -> body

"something".foo b a

// Spacing between the dot doesn't matter
"something" . foo b a
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
