# Blocks

A block is anything between braces ie: `{}`

The synax is:

```
{ ...grammar.expression }
```

Internally, they're just expressing function with a particular<a href="">scope</a>

## Forms of blocks

They're 3 forms of blocks.

_body blocks_ like the kind you see in function declarations

```
fn -> { // body block}
```

_type blocks_ like those you see in custom type declarations

```
t = type { field: type}
```

_interface blocks_ like those you see in interface declarations
<sub>read more about <a>Interfaces</a></sub>

```
i = interface { fn: sig }
```

Context is what determines the different forms of blocks.

### Body by default

_lang_ will default to
_body block_ when the context is ambiguous

For example

```
{
  x:int = 42
}
```

This could be "create a type with a field named
_x_ of type int and a value of 42" or it could be "declare a variable _x_ of
type _int_ with a value of 42".

Because there's not enough context to determine what's happening with the block
(there's no prefixed function). _lang_ will assume you just want to create a
body block to scope variables in this case.

This is the case if there's no assigment as well:

```
{
  x:int
}
```

This is just as ambiguous as the one with the assignment.

### Comma for context

When the parser sees a comma after an expression followed by another
expression (as long as it's not the last expression) then it's a safe bet that a
_type block_ is being declared

```
{
  x:int,
  y:int,
  z:int
}
```

The interpreter will classify that as a type block because body blocks don't have
commas as expression terminators

But if the last expression has a comma in it, then it's a body bock since this
means that the function is returning multiple things

```
{
  x:int,
  y:int,
  z:int, all:point
}
```

that one **is** a body block


If we remove the commas, then _lang_ will think you're just declaring variables
and pick body block

```
{
  x:int
  y:int
  z:int
}
```

You can configure the interpreter to dis-allow empty variables like this using
`config.no_empty_variables`


### The assignment operator

Using the assignment operator `=` in a body can also provide context to the
interpreter.

A type block can only have the assignment operator in one place and that's after
type declaration on a field some. So if the assignment operator appears before
the colon in an expression, the the interpreter will determine that we're
wanting a body block

```
// Body block
{
  x = 42
}
```

Once the interpreter makes a decision on the type of block in play, you can't change it later

```
x = {
  // Ambiguous
  x:int = 44,
  y:string,
  foo:'bar',

  // The following line will tell the interpreter that this is a body block
  y = 'something else',
}

So if you try to pass it as a type block _lang_ will yell at you

y = type x

// Error: Incorrect block type. The function `type` doesn't allow ....
```

## Lock it down

If you pass a block to a function that declares what type of block it wants,
then _lang_ won't do any guessing and instead do compile checking

For example here's the <a>type</a> function declaration

```
type = grammar.type_block -> {
  // implementation
}
```

This tells the interpreter that the `type` function can only ever receive a type
block and thus when the interpreter parses code that uses it, it will only ever
look for type blocks and clear up ambiguous code

```
// This is okay
data = type {
  x: 32
}

// This is okay as well
data = type {
  x:int = 32
}

// This is not okay
data = type { 
  x:int,
  y = 4
}

// Error: Incorrect block type. The function `type` doesn't allow ....
```
