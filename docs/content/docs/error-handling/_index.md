---
title: "Error Handling"
---

Do or do not, there is no try

# Error Handling

_lang_ (by design) does not have a concept of "try" or "cathing" errors in the
traditional sense. It instead nudges developers to write code that:

- uses types to handle cases (`either`, `maybe`)
- return tuple types with errors built in
- implict `try` (attaching handlers)

## Using types


### Maybe

_lang_ provides a handful of types common in other functional languages that
allow for expressing code that may fail those are:

- `maybe`
- `either`
- `just`
- `nothing`

You can express code that returns these types to reduce the need for catching
every possible type of error

For example let's say we're going to make a call to some unreliable api and
then we're going to do `something` with the data
```
something = d ->
  // do something with data


// This is the piece that can fal
data = http.get https://api.janky.com/status

// but here we're making the assumption that we got something back
// and that janky.com was up and allowing trafic
// and that our network was stable
// and... a whole bunch of other things
something data
```

In _lang_ you would you would break up your problem by using a function and a
type instead and handle each _case_ separately:

- where it _just_ works and you get what you expect
- or it breaks and you get _nothing_ back

```
something = d -> <body>


get_data -> maybe
  http.get https://api.janky.com/status


data = do get_data

// now data is of type `maybe`, so we can handle the cases as we see fit. Here
// we use the built `match_type`
match_type data {
  just: something data
  nothing: // handle error
}
```

### Shorthand

If we look at the `get_data` function, it looks like a mandatory wrapper to just
"type cast" some value; _lang_ provides a shorthand for that by just declaring
types as `maybes` instead:

```
// instead of this
get_data -> maybe
  http.get https://api.janky.com/status

data = get_data


// we can just write
data:maybe = http.get https://api.janky.com/status
```

## Using multiple returns

_lang_ also allows for errors to be passed around by returning tuples instead.
We'll use the sample example

```
get_data -> (struct, error)
  d:maybe = http.get https://api.janky.com/status
  match_type d {
    just: d,
    nothing: error "Could not get data"
  }

data, err = (get_data)

// Either `data` or `err` will be null but not both.

unless err
  something data
  sys.exit

crash err
```

### Result

You'll notice that the multiple error approach is basically wrapping the call
to maybe. This is very common in _lang_ so there's a type that does just that,
the `result` type.
The `result` type function looks like this

```
result = type {maybe, err:error}
```

It takes in a `maybe`, and an error message and returns a struct that has 2
keys `ok` and `err`; where `ok` matches the `type` of the `just` of the
_maybe_.

```
get_data -> result
  d:maybe = http.get https://api.janky.com/status
  result d, "Could not get data"

// Then when we use it, we can know what we have
data = (get_data)
data.err
data.ok

// Remember that we can destructure a struct
ok, err = data
```

If we wanted to keep the original signature that's easily doable by using
the ellipses syntax instead

```
get_data -> (struct, error)
  d:maybe = http.get https://api.janky.com/status
  ...result d, "Could not get data"

// and know we're back to where we started
data, err = (get_data)
```

## Handling error events

The last way to handle errors is using the `on_*` events.

In a nutshell _lang_ has a concept of events that every function can _emit_,
one of these events is the `on_error` event which is called when any function
`crashes` and the function (if any) bound to said function it is passed the
`error`.

```
fn ->
  do something_that_may_break


fn.on_error = err ->
  // handle errors
```

Here we're binding a function directly to the `fn`, however we can also use any
function that has the error handling signature:

```
binding_name = a:error -> <body>
```

so this is also perfectly valid

```
my_error_handler = e:error ->
  json e | to_db
  stderr e

op1 ->
  // do something that may break

op2 ->
  // do another something that may break

op3 ->
  // and another

op4 ->
  // and so on


[op1, op2, op3, op4].every (f ->
  f.on_error = my_error_handler
)
```

### with

Attaching functions to `events` is a very common thing in _lang_, the `with`
function can be used to bind handlers to all functions in it's body

It looks like this
```
with = (handlers:struct, branch:expression) -> <body>
```

The handlers struct takes _keys_ with matching events names and _values_ as the
handlers

```
my_error_handler = e:error ->
  json e | to_db
  stderr e

with { on_error:my_error_handler } do
  op1
  op2
  op3
  op4
```

Or if you give your function the same names as the event, you can use the
shorthand

```
on_error = e:error ->
  json e | to_db
  stderr e

with { on_error } do
  op1
  op2
  op3
  op4
```
