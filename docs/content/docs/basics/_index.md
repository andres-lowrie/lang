---
weight: 1
---

# Built in types

## Numbers

They should work as expected and the compiler should do the work of figuring out
what type you intended

```
1         => Integer
1.4       => Float
1_000_000 => Integer
1/3       => Float

// You should always have the final say though
50:double => Double
```

## String


Single, Double, or use back-ticks for literal

```
"hello"         => "hello"
'hello "world"' => 'hello "world"'
`hello 
world`          => 'hello \n world'
```

When dealing with shell commands however, strings become a huge pain point; The
core library should have functions to help with this


```
// From the explicit ones
escapeDoubleQuote 'No "double quote please"'
-> 'No \"double quote please\"

// To ones that guess
escape "Escape 'double quote'"
-> "Escape \'double quote\'"

escape `Some string that's a 'mix' of "strings"`
-> `Some string that\'s a \'mix\' of \"strings\"`
```

## Booleans

```
true => true
false => false
```

## Operators

### The usual suspects

```
+ => add 
- => substract
/ => divide
* => multiply
% => modulo
^ => power
```

### Conditional

there's the more declarative ones as well

```
not => true when false
or  => true when either
and => true when both
```

## Comments

There's only the double forward slash

```
// This is it
```

There isn't a block comment. I would venture a guess that most editors these
days have a shortcut to prefix a line with comments anyway

```
// No block
// comments
// just double slash
```
