---
title: Expressiveness
---

# Expressiveness

Being able to "just declare and express" things is a staple of _lang_

## Assign if "empty"

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

However the dot `.` operator is used to allow users to both access and create arbitrarily nested structures

### Accessing Structures

Suppose we have something like this
```
person = {
  first_name,
  last_name,
  addresses: [{}, {}],
  immediate_relatives: {
    father: {},
    mother: {},
    siblings: [{}, {}],
  },
  distant_relatives: {}
}
```

We can "drill down" to the property we want by simply spamming the `dot (.)` until we're at the level we want

```
// supposing that this person's last ordered sibling is their sister
sister = person.immediate_relatives.siblings[-1]
```

### Creating/Modifying Structures

You can also add fields with the dot operator and have the assignment work the
same way it does in any other place

```
person = { first: 'Tony', last: 'Stark' }

person.occupation = 'Superhero'
person.alias = 'Iron Man'
person.weapons = ['missles', 'powerbeam', 'lazers', 'machine guns']
```

You can go arbitrarily deep as well when modifying a structure:
```
person = { first: 'Tony', last: 'Stark' }

person.friends_group = {
  avengers: [hulk, cpt_america, black_widow, hawkeye, thor],
  appears_in: {
    marvel_comics: [...],
    marvel_movies: [...]
  }
}
```

#### Self Referencing Structures

Sometimes when you're creating a structure you want to reference the structure itself, perhaps to access properties with less typing; to do so refer to the variable name the structure is being assigned to
```
person = {
  first: 'Tony',
  last: 'Stark',
  occupation: 'Superhero',
  alias: 'Iron Man',
  weapons:  ['missles', 'powerbeam', 'lazers', 'machine guns'],
  favorite_weapon: person.weapons[2]
}
```

> Note that this only works with structures that have a variable name binding and you can't self reference an "anonymous structure"


### Calling functions

As mentioned before, _lang_ doesn't have the concept of objects but the "magical dot" allows for _lang_ to shift around functions and parameters to help make code easier to read and express

The dot operator has a signature that looks some what looks like this
```
.:infix = (a, b:either:[collection,fn], ...args) -> b ...(args + a)
```

Which allows us to express the parameters a function should be applied to in reverse order. This is particularly useful when applying functions to iterables; the canonical example is filtering a list

Suppose we have a function `filter` defined somewhere:
```
filter = (pred, shapes:list:struct) -> // implementation
```

Then we could apply that function to some parameters in one two ways:

The typical way:

```
my_list = [1, 2, 3, 4]

filter (n) -> n % 2, my_list
```

or, using the `dot (.)` syntax

```
my_list = [1, 2, 3, 4]

my_list.filter (n) -> n % 2
```

when we compare the two:
```
filter (n) -> n % 2, my_list
my_list.filter (n) -> n % 2
```

While the `dot (.)` syntax will technically allow you to create a _pipeline_ of transformations via "function chaining" (other languages might call this _"method chaining"_), note that you have to enclose each function in paraenesis to help the parser decide the right thing:

```
my_list = [0..99]

// Technically works
res = my_list.(filter (n) -> n % 2).(filter (n) -> n % 4)

// But multiple lines will cause a compile warning
// suggestioning alternate function in order to help with clarity
res = my_list
  .(filter (n) -> n % 2)
  .(filter (n) -> n % 4)
  .(filter (n) ->
    n + 2
    n 2 4
    n)
  .(filter)
=> Compile warning: Instead of function chaning, consider the `pipe` ...
```


_lang_ offers an alternate way of creating pipelines like this.

## Pipe

The pipe operator `|` can be used to pass the output of one function as input to
another function

```
my_list = [0..99]

// Single line when easy
rest = my_list | filter (n) -> n % 2 | filter (n) -> n % 4

// Or multiple lines when many functions are at play
rest = my_list
  | filter (n) -> n % 2
  | filter (n) -> n % 4
```
