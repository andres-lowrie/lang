---
title: "Concurrency"
---

# Concurrency Methods

_lang_ has some simple concurrency abilites built into it

## async/await

By default everything in _lang_ runs in the main thread synchronously. You can
tell _lang_ to run a function concurrently using the `async` function:

```
print_something -> async do
  sleep 2
  stdout 'third'

stdout 'first'
(print_something)
stdout 'second'

=> 'first'
=> 'second'
=> 'third'
```

You can make any function asynchronous by just passing it to `async`

```
print_something -> sleep 2 stdout 'third'

new_print = async print_something

stdout 'first'
(new_print)
stdout 'second'

=> 'first'
=> 'second'
=> 'third'
```

You can use the `await` function to denote that you want to block the thread
until that value comes back

```
print_something -> async (->sleep 2 stdout 'third')

stdout 'first'
x = await print_something
stdout 'second'

=> 'first'
=> 'third'
=> 'second'
```

Both `async/await` can actually take either a function or a list of functions
as their parameter and when a list is passed _lang_ will run all the functions
in the list at once on separate threads

```
a = async <body>
b = async <body>
c = async <body>

// Don't wait for anything
async [a, b, c]


// Wait for all values
list_of_values = await [a, b, c]
```

## queue

If you want two functions to communicate with each other, you can use a

`queue`

```
q = queue

// First you setup a handler to something everytime data arrives
q.sub = (data ->
  // Do something with "data"
)

// Then you can send messages into the queue
every (i -> q.pub "Sending message {i}") [1..10]
```

The above code _may_ exit before the "subscribe" function runs for every
message however you can use some of the built-in queue functions to wait for
events however if you want to keep the queue alive,

For example:

Wait for user to type in `ctrl-c`:

```
q = queue

q.sub = <body>

q.pub <body>

q.until keyboard.interrupt
```

Stop on errors:

```
q = queue

q.sub = <body>

q.pub <body>

// Stop on any error
q.on_error q.stop

// Stop on error from the subscriber or publisher
q.sub.on_error q.stop
q.pub.on_error q.stop
```

You can also stop the queue from the publisher or subscriber as well

```
// Create a queue that has a 50/50 shot at just stopping
q = queue

q.sub = data ->
  if rand.bool q.stop

q.pub = data ->
  if rand.bool q.stop

// You can use the `on_stop` event and bind a handler to hook into the stop
// event
q.on_stop <body>
```

You can also have _lang_ ensure that the messages in the queue are of a type as
well

```
q:queue:int = queue
q:queue:<your fn here> = queue
```

## Concurrency configuration settings

todo
