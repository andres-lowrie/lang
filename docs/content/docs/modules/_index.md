---
title: "Modules"
---

# Modules

A module is a file that has functions defined in it.

```
// A file named `math`

add = a b -> a + b
```

To use it in your code, you can use the `import` function:

```
import math

math.add 1 2
// => 3
```

## Public and Private functions

By default every function defined is "public" and will be exported. You can
stop functions from being exported by using the `priv` function: 

```
// A file named `math`

add = a b -> a + b
priv sub = a b -> a - b



// Another file
import math

math.sub 1 2 
// => compile error: no function named `sub` in the `math` module ....
```

You can be excplicit if you like and use the `pub` function as well:

```
// A file named `math`

pub add = a b -> a + b
priv sub = a b -> a - b
```

You can also invert the behavior of modules if it makes sense for your project
and make everything private by default and force people to explicitly make
things public instead:

```
// A file named `math`

config.module.priv_by_default {
  pub add = a b -> a + b
  sub = a b -> a - b
}

math.sub 1 2 
// => compile error: no function named `sub` in the `math` module ....
```

## The `import` function

The function definition looks like this:

```
import = ...path:string -> {}
```

Meaning that anything that resolves to a string can be used meaning that you
can have dynamic style imports:

```
import (funcs_ + os.name)
```

If you want to import a module that's outside of the module path, you can do that as well:

```
import /some/file/someplace/else
```

More about <a>Import Paths</a>

### namespacing magic

You can control the namespace of the functions you explicitly import pass the string `as` to import

```
import math as m

// ....
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
// => 3.14

m.geometry.pi
// => 3.1415926535897932384626433

m.pi
// => Runtime Error: Could not find function 'pi' in 'm'....
```

### destructuring

You can also pull out functions from a module using the destructuring syntax:

```
pi, add = import math
```
