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
something = something
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

See [Types]() for more

## Unpacking

You can also "pull out" multiple values from things.

There are a couple of ways to do this the first is

### Pattern match style

Match things on the left side of the `=` with things from the right

```
a, b, c = 1, 2, 3
```

You can also ignore things by using the underscore `_` in the left
hand side:

```
a, _, c = 1, 2, 3

a
=> 1
c
=> 3
```

In the above examples, the _thing_ to the right of the `=` are 3 distinct "numbers"; note however that it could be "strings"

```
a, b, c = "foo", "bar", "baz"
```

or functions

```
a, b, c = fn1, fn2, f3
```

or the results 3 functions

```
a, b, c = (fn1), (fn2), (f3)
```

The rule for the pattern matching is that the assignment will "match up" anything separated by a comma on the left of the `=` to its "partner" from the right side

```
// No need to add a comma when its just 1 thing on the right side
one = 1

// Things line up
one, two = 1, 2

// If things don't line up then an error is thrown. Two commas on the left
// but only a single one on the right
one, two, three = 1, 2
=> Compile error: Could not match all values on line ...

```

The thing to remember with this style is that **the commas need to line up**

Another way to pull out values is by _destructuring_ complex types

for example

### Lists

```
data = [1, 2, 3]

// Infer type
a, b, c = data

// Explicit type
a:int, b:int, c:int = data
```

You can group things from a list into smaller lists by using the ellipsis (called _"capturing"_) 

```
data = [1, 2, 3, 4, 5, 6]

// capture from the back
a, b, c, ...rest = data
=> a, b, c
=> 1 2 3

=> rest
=> [4, 5, 6]

// capture from the front
[...front, last] = data
=> front
=> [1, 2, 3, 4, 5]

=> last
=> 6

// capture from the middle
[front, ...middle, back] = data

=> front
=> 1

=> middle
=> [2, 3, 4, 5]

=> rest
=> 6
```

### Structures

When pulling out variables from a structure, you need to name the variables the
same thing they're named in the structure

```
data = { prop_a: 'a', prop_b: 'b' }

// Infer type
prop_a, prop_b = data

// Strict type
prop_a:string, prob_b:string = data

// The following will not work
a, b = data
=> Runtime error: The structure `data` doesn't have a field `a` ....

```


### Functions

If a function returns an iterable type (eg: lists, structures, tuple) you can assign
results to multiple variables

```
// Here's a function that returns a tuple
nautical -> 'port', 'starboard'

// We can now pull out the values into separate values
left, right = nautical

=> left
=> 'port'

=> right
=> 'starboard'
```
