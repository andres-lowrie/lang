---
title: "First Class Commands"
---

# First Class Commands

_lang_ allows scripts to call "cli commands" directly with no ceremony:

```
on_the_box = who

stdout on_the_box
=> "jane console Mar 4 21:33\n jane ttys001 Mar 10 15:06"
```

You can also do the piped style commands

```
ls
=> "README.md \ndocs"

readme = ls | rg *.md

stdout readme
=> "README.md"
```

This is because `|` in _lang_ is the [pipe]() operator.

_lang_ supports types which means that you can run commands and get things out
that are not just "text":

```
import strings


// What you would expect
x = fd --type d '.' docs/content

stdout x
=> "docs/content/docs\ndocs/content/basics\ndocs/content/....


// coerce into types instead
x = split strings.newline (fd --type d '.' docs/content)

stdout x
=> [ "docs/content/docs", "docs/content/docs/basics", "docs/c...


// Express what you want to express
x = split strings.newline (fd --type d '.' docs/content)
  | first
  | split '/'
  | last
  | tr a-z A-Z

stdout x
=> "DOCS"
```

This works because a command is just a function and thus you can use commands
like any other function in _lang_ and leverage its features

```
import sys

split_words = import strings

// Here we make a partially applied function out of the grep command
g = grep -irn

// Grab a random word from a line ending in a random letter
pattern = (rand a..z) + $
lines = split_words (g pattern sys.cwd)
pos = ((a, b) -> rand b a) (len lines)

stdout lines[pos 0]
```

## Environment Variables

You can read and set environment variables from using the `sys.env` function

```
import sys/env

// Read
user = env 'user'

// Set
env 'user' Jack
```

## Standard Error and Standard Out

By default, when _lang_ determines you're using a command as a function, you
can use [multiple returns]() to capture output and error:

```
out, err = bogus

out
=> Nothing

err
=> "zsh: command not found: bogus"
```

The multiple outputs that a command will return are:

1. Standard Output
2. Standard Error
3. Return Code

```
out, err, code = bogus

out
=> Nothing

err
=> "zsh: command not found: bogus"

code
=> 127
```

# Commands are just functions

When you call a command in a script, _lang_ will look for it in the scope
that's currently available. If it doesn't find it, it will then look for the
command in the `PATH` set in the runtime environment.

If it finds it there, it will then create a function on the fly like this:

```
name_of_command_it_found = ...a:str -> (str, str, int); <body>
```

Which will allow it to act like any other function

More about how the [lookup scope]() works

## The magical cli arguments functions

You'll notice that in the above examples the use of the aseterix `*` and hyphen
`-` and slashes `/` but _lang_ isn't doing math, no multiplicaton , subtraction
or division

That's because operators in _lang_  are functions are called with by spacing
out each argument, thus when a command is run say `ls` the flags and options
becomes part of the argument:

```
// `-al` is being sent as the string `-al`
ls -al '.' 


// Here the space between the hyphen is seen as the subtract infix function
// this will crash
ls - al '.'
```

So, when using hyphens, slashes, and asterisks special care has to be taken
to mind the spacing between values:

```
some/path/here      => a str
some / path / here  => a Compile time Error (unless those variables are numbers)
*sup                => a str
* sup               => a partially applied function
```

**What about negative numbers?**

That's the trade-off, _lang_ doesn't support expressing negative numbers
"literally", calling

```
1 + -1

// will result in a concated string
=> str "1-1"
```


Instead use the `math` module which has a `neg` function that will return the
negative value of the number passed to it:

```
1 - neg 1
=> 2
```
