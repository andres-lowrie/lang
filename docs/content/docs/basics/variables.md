---
weight: 2
---

# Variables

## Assignment

Very little ceremony when it comes to variable assigning

```
foo = 1             => foo is the number 1
foo = 'something'   => foo is the string 'something'
foo = { bar: 'bas'} => foo is a structure
foo = a b -> a + b  => foo is a function
```

Basically it's 

```
anything = anything
```

## Strictness

_lang_ will infer types by default, but you always get the last word

```
foo:int = 42
```

See <a>Types</a> for more

## Destructuring

You can also destructure stuff

### Pattern match style

Match things on the left side of the `=` with things from the right

```
a, b, c = 1 2 3

// or 

a, b, c = 'a' 'b' 'c'
```

### Structures

When pulling out variables from a structure, you need to name the variables the
same thing they're named in the structure

```
data = {a: 'a', b: 'b'}

// Infer type
a, b = data

// Strict type
a:string, b:string = data

// The following will not work
a, c = data
=> Runtime error: The structure `data` doesn't have a field `c` ....

```

### Lists

When destructuring lists however, you can name the variables anything you want

```
data = [1, 2, 3]

// Infer type
a, b, c = data

// Explicit type
a:int, b:int, c:int = data
```

You can also either ignore the things in the list you don't care about or
capture them as a list in a new varialbe

```
data = [1, 2, 3, 4, 5, 6, 7]

a, b , c, ...rest = data

=> a 1
=> b 1
=> c 1
=> rest [4, 5, 6, 7]
```

### Functions

If a function returns multiple things you can assign results to
multiple variables

```
nautical -> 'port', 'starboard'

left, right = nautical

=> left 'port'
=> right 'starboard'
```
