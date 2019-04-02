---
title: "Concurrency"
---

# Concurrency Methods

_lang_ has some simple concurrency abilites built into it

## async

By default everything in _lang_ runs in the main thread synchronously. You can
tell _lang_ to run a function concurrently using the `async` function:

```
print_something -> async {
  sleep 2
  stdout 'third'
}

stdout 'first'
print_something
stdout 'second'

=> 'first'
=> 'second'
=> 'third'
```

If you assign a value to an `async` function _lang_ will think you want to wait
for the value to come back:

```
print_something -> async (-> sleep 2 stdout 'third')

stdout 'first'
x = print_something
stdout 'second'

dump x

=> 'first'
=> 'third'
=> 'second'

```

`async` can acutally take either a function or a list of functions as its
parameter and when a list is passed _lang_ will run all the functions in the
list at once.

If you want to wait for all of them to return something, then just assign
something to the return of async


```
a = async { ... }
b = async { ... }
c = async { ... }

// Don't wait for anything
async [a, b, c]


// Wait for values
list_of_values = async [a, b, c]
```

## queue

If you want two processess to communicate with each other, you can use a
`queue`

```
q = queue

// First you setup a handler to something everytime data arrives
q.sub = (data -> {
  // Do something with "data"
})

Then you can send messages into the queue
every [1..10] (i -> q.pub "Sending message {{i}}")
```

The above code will exit before the "subscribe" function runs for every
message however you can use some of the built-in queue functions to wait for
events however if you want to keep the queue alive,

For example:

Wait for user to type in `ctrl-c`:

```
q = queue

q.sub = {...}

q.pub ...

q.until keyboard.interrupt
```

Stop on errors:

```
q = queue

q.sub = {...}

q.pub ...

// Any error
q.on_error q.stop

// Error on the subscriber / publisher
q.sub.on_error q.stop
q.pub.on_error q.stop
```

You can also stop the queue from the publisher or subscriber as well

```
// Create a queue that has a 50/50 shot at just stopping
q = queue

q.sub = data -> {
  if rand.bool q.stop
}

q.pub = data -> {
  if rand.bool q.stop
}

// You can use the `on_stop` handler to hook into the stop event
q.on_stop {...}
```

You can also have _lang_ ensure that the messages in the queue are of a type as
well

```
q:queue:int = queue
q:queue:<your fn here> = queue
```

## Concurrency configuration settings

todo
