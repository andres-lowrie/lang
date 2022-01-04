---
title: "Modules"
---

# Modules

A module is a file that has functions defined in it.

```
// A file named `math`

add = (a, b) -> a + b
```

To use it in your code, you can use the `import` function:

```
import math

math.add 1 2
=> 3
```

## Public and Private functions

By default every function defined is "public" and will be exported. You can stop functions from being exported by using the `priv` function and passing in the functions you want to hide:

```
// A file named `math`

add = (a, b) -> a + b
sub = (a, b) -> a - b

// You can inline it 
div = priv (a, b) -> a / b
mul = priv (a, b) -> a * b

// Or you can declare them all at once
priv div, mul



// Another file
import math

math.mul 1 2
=> Import error: no function named `mul` in the `math` module ....
```

If you want to switch the default and say that all functions should be private unless defined otherwise, you can use the `pub` function explicitly:

```
// A file named `math`

add = pub (a, b) -> a + b
sub = pub (a, b) -> a - b
div = (a, b) -> a / b
mul = (a, b) -> a * b

// or all at once
pub
  add
  sub
```


You can either use the `pub` or `priv` functions but not both, doing so will cause an error

```
// A file named `math`

add = (a, b) -> a + b
sub = (a, b) -> a - b
div = (a, b) -> a / b
mul = (a, b) -> a * b

priv div, mul

pub add, sub

=> Import error: module at `math` uses both `pub` and `priv` this is...
```

## The `import` function

### basics

You basically call the function at the top of your file

```
import mod1
```

You can specify many imports by calling the function a bunch of times

```
import mod1
import mod2
import mod3
```

`import` is a _variadic_ function so you can also do:

```
import (mod1, mod2, mod3)

// or using multiline application
import
  mod1
  mod2
  mod3
```

You can also mix and match:

```
import mod1
import mod2
import mod3

import
 mod4
 mod5
 mod6
```

## local modules

There are 3 types of "local" modules in _lang_.

### project modules

The first and most common are those that are local to your project. To import
those modules you point to the name of a file.

Suppose we have a project like this

```
├─ lib/math
├─ lib/mod2/otherthing
└─ src/
   ├─ component/stuff
   ├─ mod1
   └─ main

where `main` is the entrypoint for _lang_
```

To import `mod1` I would do:

```
import mod1
```

to import `stuff` I would do:

```
import component/stuff
```

To import from the local libraries I would do

```
import math
import mod2/otherthing
```

The folders `src/` and `lib/` are implied by _lang_ so you don't need to
specify them. The differene between those 2 locations comes down to which
directory _lang_ will check in first (spoiler it's `lib`)

### system modules

The second type of local modules are those that come from _lang_ itself

When you do

```
import math
```

_lang_ will look for a file named _"math"_ in this order:

- first in the directory `{lang_install_location}/lib/`
- then in the directory `lib/`
- then in the directory `lib/third-party/`
- then in the directory `src/`

if it doesn't find it in any of these locations you'll get an import time
error.

### absolute path

The other type of local modules are those you point to specifically using a path

```
import /home/user/mystuff/math
```

This will bypass the default behavior and look for this file directly

## external modules

You can also import modules from the web:

```
import github.com/user/repo/lib/math

// or
import ssh://git@github.com:user/repo/lib/math

// or
import github.com/user/repo/lib/math#commit-ish

// or
import domain.com/langstuff/math?k=v

// or
import ssh://user@host:/home/user/stuff/math

// or
import s3://bucket/filepath/math

// or
import nfs://server/filepath/math

// and a bunch other protocols too, just as long as the content is a valid lang
// module
```

[link to import protocol setup]()

## Dynamic Imports

The function definition for `import` looks like this:

```
import = ...path:str -> // implementation
```

Meaning that anything that resolves to a string can be used as an import site.
For example, say you want to only import the functions you've tested for the
operating system the user is currently on:

```
import (funcs_ + os.name)
```

Or if you only want to import things under certain conditions:
```
if feature_flag_set
  import some/mod; mod.do_something
```

Or if you want to let the user decide, you can do that as well:

```
user_path = prompt "Where are your modules at?"
import user_path
```

> Using import at run time like this does come with some gotchas [read more about them here]()

## Namespacing

You can control the namespace of the functions imported by assigning them to variable:

```
m = import math
...
```

You can also combine different modules into a "namespace" if you like:

```
m = import
  math
  geometry
  trigonometry
  github.com/cool-lib/mod


// Public functions from those modules are now available on the `m` structure
m.add
m.area
m.cos
m.cool_lib_fn
```

When combining functions via import, _lang_ will resolve conflicts by embedding
the functions into a structure named the same as the modules that are colliding:

```
// file named `math`
pi = 3.14


// file named `geometry`
pi = 3.1415926535897932384626433


// Some other file
m = import math, geometry, trigonometry

m.math.pi
=> 3.14

m.geometry.pi
=> 3.1415926535897932384626433

m.pi
=> Runtime error: Could not find function 'pi' in any of the modules in `m`....
```

## Destructuring

You can also pull out functions from a module using the destructuring syntax:

```
pi, add = import math
```

> Note this follows the rules for [unpacking a structure]()
