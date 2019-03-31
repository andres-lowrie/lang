---
title: "First Class Commands"
---

# First Class Commands

_lang_ allows scripts to call "cli commands" directly with no ceremony:

```
on_the_box = who

stdout on_the_box
// => "jane console Mar 4 21:33\n jane ttys001 Mar 10 15:06"
```

You can also do the piped style commands

```
ls
// => "README.md \ndocs"

readme = ls | rg *.md

stdout readme
// => "README.md"
```

_lang_ supports types which means that you can run commands and get things out that are not just "text":

```
// What you would expect
x = fd --type d . 'docs/content'

stdout x
// => "docs/content/docs\ndocs/content/basics\ndocs/content/....


// coerce into types instead
x = split strings.newline fd --type d . 'docs/content'

stdout x
// => [ "docs/content/docs", "docs/content/docs/basics", "docs/c...


// Express what you want to express
x = split strings.newline fd --type d . 'docs/content'
  | first
  | split /
  | last
  | tr a-z A-Z

stdout x
// => "DOCS"
```

This works because a command is just a function and thus you can use commands
like any other function in _lang_ and leverage the features of _lang_ to run
commands


```
g = grep -irn

// Grab a random word from a line ending in a random letter
pattern = (rand a..z) + $
lines = split_words (g pattern cwd)
pos = (a b -> rand b a) len lines


stdout lines[pos 0]
```

## Environment Variables

You can read and set environment variables from using the `env` function

```
// Read
user = env 'user'

// Set
env 'user' Jack
```

## Standard Error and Standard Out

By default, when _lang_ determines you're using a command as a function, you
can use <a>multiple returns</a> to capture output and error:

```
out, err = bogus

out
// => Nothing

err
// => "zsh: command not found: bogus"
```

The multiple outputs that a command will return are:

1. Standard Output
2. Standard Error
3. Return Code

```
out, err, code = bogus

out
// => Nothing

err
// => "zsh: command not found: bogus"

code
// => 127
```



You can disable the multiple returns with a config block which will
cause the return to just be the return code

```
config.no.multi_return_command { 
  bogus
  // => 127
}
```

# Commands are just functions

When you call a command in a script, _lang_ will look for it in the scope
that's currently available. If it doesn't find it, it will then look for the
command in the `PATH` set in the runtime environment.

If it finds it there, it will then create a function on the fly like this:

```
name_of_command_it_found = ...a:string -> string, string, int  {}
```

Which will allow it to act like any other function

More about how the <a> lookup scope</a> works

## The magical cli operator functions 

You'll notice that in the above examples the use of the aseterix `*` and hyphen
`-` but _lang_ isn't doing math, no multiplicaton or subtraction

That's because those functions are defined to be syntax aware like this

```
pub - = (->
  t -> not any [-, grammar.space]
  l:t, r:t -> sub l r
)
```

This allows for the hyphen to double as the string character `-` and the
subtract function based on how it appears in the script

For example:

```
// Here the hyphen has a space on either side of it and so _lang_ will call the
// `sub` function and it will do what you expect
1 - 1
=> 0


// However this will try to call the function `1` with the string argument `-1`
// which will result in an error
1 -1
=> Runtime Error: The function `1` does not take arguments....


// The same applies for functions that have been bound to variables
a -> 1
b -> 1

a - b
=> 0


a- b
=> Runtime Error: No function named `a-` found in module....
```

**What about negative numbers?**

The `math` module has a `neg` function that will return the negative
value of the number passed to it:

```
1 - neg 1
=> 2
```

You can disable this via the config block `config.cli.no.auto_flags`
