---
title: Introduction
type: docs
---

# Language

## It should

- It should be easy to read
  - I'm super inspired by `coffeescript` and `python`. Like if those 2 languages
    had a baby...and that baby was a functional language
- Be ultra configurable and extensible at **every** point
  - ^ this may be a horrible idea... perhaps what I want are _macros_ idk... let
    it ride
  - my intent is to allow users to extend the language and have that extending
    be a first class citizen
- Be functionally declarative
- Have commands and the cli be first class citizens...but skip the arcane
  invocation part (reduce the amount of weird symbols and shit)
- Be as strict as you make it but guess everywhere else
  - I mean optionally typed
  - and implicit interfaces <-- what I think is the best feature of `golang`
- Allow testing to be a first class citizen while still letting the code say what
  it wants to say (tests should bend to code not the other way around)
- Stuff should just be included and it should eliminate yak-shaving
  - There should only be 1 formatter and style: a la go-fmt... no dicussions ever...period
  - There should only be 1 package manager
  - There should only be 1 testing framework
  - There should only be 1 build/task running system
  - and all of these should all be built-in (ships with the language)
