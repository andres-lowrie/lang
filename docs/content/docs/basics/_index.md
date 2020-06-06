---
title: "Basics"
---

# Built in types

## Numbers

They should work as expected and the compiler should do the work of figuring out
what type you intended

```
1         => int
1.4       => float
1_000_000 => int
1/3       => float

// You should always have the final say though
50:double => double
```

## String


Single or double for typical stuff; and back-ticks for literals

```
"hello"          => "hello"
'hello "world"'  => 'hello "world"'
`hello \n world` => 'hello \n world'
```

When dealing with shell commands however, strings become a huge pita; The
core library should have functions to help with this


```
// From the explicit ones
escape_double_quote 'No "double quote please"'
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
! => not
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
