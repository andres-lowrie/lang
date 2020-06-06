# Variables

## Assignment

Very little ceremony when it comes to variable assigning

```
foo = 1                => foo is the number 1
foo = 'something'      => foo is the string 'something'
foo = { bar: 'bas'}    => foo is a structure
foo = (a, b) -> a + b  => foo is a function
```

Basically it's 

```
anything = anything
```

## Strictness

_lang_ will infer types by default, but you always get the last word

```
foo:int = 42
=> 42

foo:float = 42
=> 42.0

foo:string = 42
=> "42"
```

See <a>Types</a> for more

## Destructuring

You can also destructure stuff

### Pattern match style

Match things on the left side of the `=` with things from the right

```
a, b, c = 1 2 3
```

You can also ignore things by using the underscore `_` in the left
hand side:

```
a, _, c = 1 2 3

a
=> 1
c
=> 3
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

You can capture things into lists by using the ellipsis

```
data = [1, 2, 3, 4, 5, 6]

// capture from the back
a, b, c, ...rest = data

dump a b c
dump rest
=> 1 2 3
=> [4, 5, 6]

// capture from the front
[...front, last] = data
dump front last
=> [1, 2, 3, 4, 5]
=> 6

// captre from the middle
[front, ...middle, back] = data

dump front
dump middle
dump rest
=> 1
=> [2, 3, 4, 5]
=> 6
```

### Functions

If a function returns iterable types (lists, structures, tuples) you can assign
results to multiple variables

```
nautical -> 'port', 'starboard'

left, right = nautical

=> left 'port'
=> right 'starboard'
```
