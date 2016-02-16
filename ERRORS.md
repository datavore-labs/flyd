# Errors in flyd
`Kefir` handles streams by seeing it as values wrapped in an event description. These descriptions partition events into `value` and `error`, and `Kefir`'s API is written with that in mind.

The goal of this is to determine how to bring functionality similar to that into `flyd`.

## Ethos
* It should not be the concern of `flyd` to handle exceptions for the user -- any `throw` should result in a hard failure.
* Silent failures are bad (current way `flyd` handles Promise.reject)
* API must follow [fantasty-land](https://github.com/fantasyland/fantasy-land)
* Unopinionated implementation as possible


## Concepts
+ The stream is of `events`
+ Each stream has a `left` and a `rigth` side
+ The right side is the domain objects
+ The left side is meta in our case errors
+ Keep the api by default operating on `right`

## The Api
`s` is a stream

### Setting data s(...) is overloaded
+ `s(value)` is the default case takes a value makes it a right and pushes it down the stream
+ `s(promise)` if the promise resolves pushes a right, otherwise pushes a left
+ `s(either)` pushes a right or left based on either.left either.right

### Getting data
+ `s()` get the last right value or throws an exception if there is a left value
+ *new `s.left()` get the last left value or throws an exception if there is a right value
+ *new `s.isRight()` and `s.isLeft()` return boolean so you know what the stream contains

### Core functions
+ `.map()` works only on rights and ignores lefts
+ *new `.mapAll()` gets all events as an `Either` or some other lightweight type defining `lefts` and `rights`
+ `.combine()` and `.merge()` stay the same they work on events
+ `.scan()` needs more thought
+ `.on()` works on `rights` only

### Move `.transduce()` to a module

### Add some helper modules
+ `.onAll()`
+ `.mapLeft()`
+ `.swap()` swaps `rights` and `lefts` - might want this in core for performance reasons
+ others ...
