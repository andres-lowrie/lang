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

By default every function defined is "public" and will be exported. You can
stop functions from being exported by using the `priv` function:

```
// A file named `math`

add = (a, b) -> a + b
sub = (a, b) -> a - b

priv sub



// Another file
import math

math.sub 1 2
=> Error: no function named `sub` in the `math` module ....
```

You can be excplicit by using the `export` function

```
// A file named `math`

add = (a, b) -> a + b
sub = (a, b) -> a - b
div = (a, b) -> a / b

export add sub
```

You can either use the implicit syntax or the explicit one but not both:

If you use the `export` and `priv` function in your module, you'll get an error

```
// A file named `math`

add = (a, b) -> a + b
sub = (a, b) -> a - b
div = (a, b) -> a / b

priv div

export add sub

=> Compile Error: module at `math` uses both `export` and `priv` this is...
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

or you can pass a _tuple_ to it:

```
import (
 mod1,
 mod2,
 mod3,
)
```

You can also mix and match:

```
import mod1
import mod2
import mod3

import (
 mod4,
 mod5,
 mod6,
)
```

### local modules

There are 3 types of "local" modules in _lang_.

#### project modules

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

To import from the local library I would do

```
import math
import mod2/otherthing
```

The folders `src/` and `lib/` are implied by _lang_ so you don't need to
specify them. The differene between those 2 locations comes down to which
directory _lang_ will check in first (spoiler it's `lib`)

#### system modules

The second type of local modules are those that come from _lang_.

When you do

```
import math
```

_lang_ will look for a file named _"math"_ in this order:

- first in the directory `{lang_install_location}/lib/`
- then in the directory `lib/`
- then in the directory `lib/third-party/`
- then in the directory `src/`

if it doesn't find it in any of these locations you'll get an compile time
error.

#### absolute path

The other type of local module are those you point to specifically using a path

```
import /home/user/mystuff/math
```

This will bypass the default behavior and look for this file directly

### external modules

You can also import modules from the web:

```
import github.com/user/repo/lib/math

// or
import domain.com/langstuff/math?k=v

// or
import ssh://user@host:/home/user/stuff/math

// or
import s3://bucket/filepath/math

// and a bunch other protocols too, just as long as the content is valid a lang
// module
```

[link to import protocol setup]()

## Dynamic Imports

The function definition for `import` looks like this:

```
import = ...path:tup:str -> <body>
```

Meaning that anything that resolves to a string can be used as an import site.
For example, say you want to only import the functions you've tested for the
operating system the user is currently on:

```
import (funcs_ + os.name)
```

If you want to import a module that's outside of the module path, you can do that as well:

### namespacing

You can control the namespace of the functions you explicitly import pass the string `as` to import

```
import math as m

...
```

You can also combine modules into a "namespace" if you like:

```
import math, geometry, trigonometry as m


// Public functions from those modules are now available on the `m` structure
m.add
m.area
m.cos
```

When combining functions via import, _lang_ will resolve conflicts by embedding
the functions into a structure named the same as the modules that are colliding:

```
// file named `math`
pi = 3.14


// file named `geometry`
pi = 3.1415926535897932384626433


// Some other file
import math, geometry, trigonometry as m

m.math.pi
=> 3.14

m.geometry.pi
=> 3.1415926535897932384626433

m.pi
=> Runtime Error: Could not find function 'pi' in 'm'....
```

### destructuring

You can also pull out functions from a module using the destructuring syntax:

```
pi, add = import math
```
