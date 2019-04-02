---
title: "Event Hooks"
---

# Event Hooks

_lang_ let's you handle errors by listening for events using multiple methods


```
fn -> {
  // something that may break
}

fn.on_error -> {
  // handle errors
}
```

You can also wrap a block with an event handler using the `with` function

```
on_error -> {
  // handle error
}

with { on_error } {
  // handle error
}
```

## hooks for `blocks`

**on\_error :** When any error happens on when the body blocks runs
