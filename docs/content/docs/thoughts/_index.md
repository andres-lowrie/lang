# thoughts

## Base Constructs

There are only 2 constructs in _lang_:

- types
- functions

Everything is built from these constructs

Types can be further broken down into 2 categories:

- primitive
- complex

The _primitive_ types are those that are provided by the operating system the interpreter is running in, but they boil down to things like (`int`, `float`, `bool`, things like that) 

_complex_ types can then span multiple things like _structures_ , _lists_, _tuple_, _function_, stuff like that

[Full list]()


----


## Type Casting

The "magical dot" allows for _lang_ to shift around functions and parameters to make code easier to read in particular when applying functions to iterables

The canonical example is filtering a list
```
my_list = [1, 2, 3, 4]

// assuming `filter` has signature like:
//
// filter:iter = (a:any, i:iter) ->
// 
// Then this is correct
filter (n) -> n % 2, my_list

// but this is easier on the eyeballs
my_list.filter (n) -> n % 2
```

However we can leverage this same concept to do casting without having to call a function at run time

In order to do type casting, we need to call some `*TypeConstructor` function and pass it the value we want to convert; the dot syntax however allows us to do that in a way that is more convenient:

```
n:int = 1

// These 2 are equivalent
x = float n
x = n.float
```


## Immutable by default

To keep things simple, _lang_  elects to be immutable by default, meaning that once variables are assigned some value, those values cannot change.

What that means in practice and how that affects type inference

Say we declare some variable without setting a type on it:

```
x = 123
```

_lang_ will take this assignment and infer the following

```
typeof x
=> x:num
```

If we then try and change the value of `x` to something else, _lang_ will complain

```
x = 123

x = 456

=> Compile error: `x` was inferred to be `x:imut:num`. If you really want...
```

The compiler error shows that _lang_ assumes that everything is immutable, you can see so by reading the "fully qualified type" in the message

As outlined by the error message, there are several ways to allow the above script to work which are:

- Using the **MutableTypeFn**
- Using the **MutableValueFn**
- Inverting the **Project Settings**

### MutableTypeFn

This one is pretty straight, you basically declare a more complex type when assigning a value:

```
x:mut = 123

x = 456
typeof x
=> x:mut:num
```

Also note that mutability only applies to values and not types:

```
x:mut = 123

x = "some string"
=> Compile error: `x` was inferred to be `x:mut:num`. If you really wa...
```

Everything works as expected, you still get a decent level of protection against things changing to a completely different type

### MutableValueFn

This method is a little different in that you basically defer the type inference till runtime, looks like this:

```
x = mut 123

x = 456
```

While this method achieves the same thing as using the type fn, _lang_ won't actually enforce this until the program is actually running.

_umm...actually_

When the type can be resolved at compile time, then _lang_ is smart enough to infer a stricter type even if the _MutableValueFn_ is used, but if the type cannot be resolved during compile time then _lang_ will be provide a much broader type

Visualizing this in the repl

```

// Here 123 can be determined to be a num
// because mut is being applied to a basic value with a resolvable type
// ie: 123 is a static value ergo _lang_ can be stricter
x = mut 123
typeof x
=> x:mut:num

x = 456
typeof x
=> x:mut:num
```

but if we were to assign a value that cannot be resolved until runtime then the type is much broader

For example, a function that we don't specify a return type for

```
some_function -> ...

x = mut some_function
typeof x
=> x:mut:any
```

Here lang can't know what `some_function` is going to return so it has to default to the _any_ type, meaning that this can literally be anything.

If we _were_ to define the return type for the function then the _compilier_ inference system would make the correct decision

```
some_function = ():num -> ...

x = mut some_function
typeof x
=> x:mut:num
```

This works because the [AST]() can be traversed and the intended type be found

### Project Settings

If you find yourself in the unfortunate situation where you need to declare a bunch of mutable variables, then you can tell _lang_ to instead default all variables to be _mutable_ instead of _immutable_

> config.lang
```
{
  vars_default_immutable: false
}
```

If we go back to our repl, we can see that the type are inferred as we set

```
x = 123
typeof x
=> x:mut:num
```

_lang_ also provides the accompanying `imut` function to allow the possibility of perhaps climbing out of hell

```
x = imut 123
typeof x
=> x:imut:num
```

`imut` and `mut` are symmetrical from a user perspective, so all the same rules outlined for `mut` apply


## Omitting commas

_lang_ is whitespace sensitive since it does some clever parsing to make many parametered function easier to call

Here's how to applies to functions

```
call_func p1, p2, p3, p4, p5, p6

// or
call_func
  p1
  p2
  p3
  p4
  p5
  p6
```

This comes in handy when dealing with function that take other functions as control strucutres, for example the `if`

```
// This is correct
if bool_fn, cond, alternate

// but this makes it easier to read
if bool_fn
  cond
  alternate
```
