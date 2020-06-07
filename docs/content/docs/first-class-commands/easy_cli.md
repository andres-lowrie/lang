# Easy Cli

_lang_ should make interacting with the cli super easy and feel as native as it
does with any shell

## Reading parameters

As expected you can access the arguments via a list of everything passed into
the program via the `argv` list:

```
stdout argv
```

Calling it like this

```
$> ./my_cli -globalflag a -b --c "d" 'e=f' g 1

=> ["-globalflat", "a", "-b", "--c", "d", "e=f", "g", 1]
```

However the real nicities come with the parsing that's built into _lang_ using `argp`; 

```
pretty argp
=>
{
  flags: {globalflag: true},
  positional: ["a", "g", 1],
  a: {
    flags: {b: true},
    parameters: {c: "d", e: "f"},
  },
  g: {
    positional: [1]
  }
}
```

`argp` is a structure that parses out argv into logical groupings of `flags`,
`positional`, and `parameters`. It has built-int support for "sub-command"
style cli calls as well in that it will created nested structures for any
positional arguments that are not numbers

The built in parsing allows you to easily define different style of cli tools.

```
// Define our functions to act like sub-commands, we'll add them to a structure
// so we can refer to using a string value
cmd.make_call = args ->
  stdout a
  stdout 'params: ' + args.parameters.keys.join ', '
  stdout 'values: ' + args.parameters.values ', '

cmd.do_something = args ->
  a = first args.positional
  stdout 'doing something: ' + a
  cmd[a] args[a]

cmd.cool = args ->
  stdout "With ask: " + args.first
  stdout "params: " + args.parameters.keys.join ', '

// Pull out the sub-command and the stuff passed to it
action = argp[0]
params = argp[action]

// And call it
cmd[action] params
```

From the command line it could look like this

```
$> ./app make_call --type x --with-thing abc

=> "make_call"
=> "params: type, with-thing"
=> "values: x, abc"


$> ./app do_something cool please --force

=> "doing something: cool"
=> "With ask: please"
=> "params: force"
```

## Writing out

Writing things out is also quite easy given the available `stdout` and `stderr` functions.

```
out, err = bogus

stdout out
stderr err
```

Writing and reading from files can be done via the `to_file` and `from_file` respectively:

```
// Read
contents = from_file /some/file/somewhere

// Write
to_file contents /another/file
```

Using the multiple returns feature, writing errors and output to different
files is also easy

```
out, err = bogus
to_file out /output
to_file err /err
```
