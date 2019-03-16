---
weight: 6
---

# Blocks

These are the most magical pieces of _lang_ since this syntax is used everywhere
and the compiler uses context to determine types

The synax is:

```
{ ...grammar.expression }
```

## Forms of blocks...down the rabbit hole

They're 2 forms of blocks.

_body blocks_ like the kind you see in function declarations

```
fn -> { // body block}
```

_type blocks_ like those you see in type declarations

```
t = type { field: type}
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

This is just as ambiguous as the one with the assignment

### The magical comma

When the compiler sees a comma after an expression followed by another
expression (as long as it's not the last expression) then it's a safe bet that a
_type block_ is being declared

```
{
  x:int,
  y:int,
  z:int
}
```

The compiler will classify that as a type block because body blocks don't have
commas as expression terminators

But if the last expression has a comma in it, then it's a body bock since this
means that the function is returning multiple things

```
{
  x:int,
  y:int,
  z:int, all:point //<= That means this is a function and therefore this should be body block
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


### The assignment operator

Using the assignment operator `=` in a body can also provide context to the
compiler.

A type block can only have the assignment operator in one place and that's after
type declaration on a field some. So if the assignment operator appears before
the colon in an expression, the the compiler will determine that we're wanting a
body block

```
// Body block
{
  x = 42
}
```

Once the compiler makes a decision on the type of block in play, you can't
change it later

```
x = {
  // Ambiguous
  x:int = 44,
  y:string,
  foo:'bar',

  // The following line will tell the compiler that this is a body block
  y = 'something else',
}

So if you try to pass it as a type block _lang_ will yell at you

y = type x

// Compile error: Incorrect block type. The function `type` doesn't allow ....
```

## Lock it down

If you pass a block to a function that declares what type of block it wants,
then _lang_ won't do any guessing and instead do compile checking

For example here's the [type]() function declaration

```
type = grammer.block:type -> type {
  // implementation
}
```

This tells the compiler that the `type` function can only ever receive a type
block and thus when the compiler parses code that uses it, it will only ever
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

// Compile error: Incorrect block type. The function `type` doesn't allow ....
```

You can be just as explicit for body blocks if you like by using the body function

```
greet = body { 
  x:string = 'hello'
  y = 'world'
  
  x + y
}

stdout greet

=> "hello world"
```
