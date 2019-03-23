
# Easy Cli

_lang_ should make interacting with the cli super easy and feel as native as it
does with any shell

## Reading parameters

As expected you can access the arguments via a list of everything passed into
the program via the `argv` list:

```
#!/bin/lang

stdout args
```

Calling it like this

```
$> ./my_cli a -b --c "d" 'e=f' g

=> ["a", "-b", "--c", "d", "e=f", "g"]
```

However the real nicities come with the parsing that's built in:

```
stdout join [args.flags, args.parameters, args.positional] \n

=> "b"
=>
=> {c: "d", e: "f"}
=>
=> ["a", "g"]
```

The built in parsing allows you to easily define different style of cli tools
easily. For example, creating the _sub-command_ style cli can be very intuitive

```
#!/bin/lang

make_call ...args -> {
  stdout a
  stdout 'params: ' + args.parameters.keys.join ', '
  stdout 'values: ' + args.parameters.values ', '
}

do_something = a -> {
  stdout 'doing something: ' + a
  a ...args
}

cool -> {
  stdout "With ask: " + args.first
  stdout "params: " + args.parameters.keys.join ', '
}

action, opts = args
action ...opts
```

Calling it could look like this

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

That being said, you can use configuration blocks to have _lang_ setup stdout
and stderr files automatically by using the config functions:
`config.stderr.to_file` and `config.stdout.to_file` respectively. This will
cause _lang_ to append output to the paths supplied to those functions
