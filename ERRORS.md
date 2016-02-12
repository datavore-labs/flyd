# Errors in flyd
`Kefir` handles streams by seeing it as values wrapped in an event description. These descriptions partition events into `value` and `error`, and `Kefir`'s API is written with that in mind.

The goal of this is to determine how to bring functionality similar to that into `flyd`.

## Ethos
* It should not be the concern of `flyd` to handle exceptions for the user -- any `throw` should result in a hard failure.
* Silent failures are bad (current way `flyd` handles Promise.reject)
* API must follow [fantasty-land](https://github.com/fantasyland/fantasy-land)
* Unopinionated implementation as possible

## API
Currently, `flyd` has the following API:
```
// Stream constructor
flyd.stream

// Stream operations
flyd.combine
flyd.merge
flyd.on
flyd.scan
flyd.transduce

// Helpers
flyd.curryN
flyd.isStream
flyd.immediate
flyd.endsOn

// Streams
var s = flyd.stream()
s()         // get the current value
s(#)        // perform an atomic update
s.end       // Access to the `end` stream of s
s.toString()

// Fantasty Land
s.of
s.ap
s.map
// does not implement s.chain
```

## Error API

The core should support the following
+ Send a promise down the stream -> if it resolves a value event is created, if it rejects an error event
+ Send an Either down the stream -> if its a left and error event is created, if its a right a value event
+ Send a value down the stream directly it becomes a value event
+ Send an error down the steam it becomes a error event
+ Get the last event (error or value)

The core combinators and access patterns should focus around values (the default case) and events the under the hood case
This is the functionality I'm thinking
+ Get the last value
+ Define map, flatmap, chain, combine, and merge on streams so that dealing with the values is easy or getting at the underlying events is easy
+ Leave out error helpers as its the off case. Helpers can be added to modules for mapError and others

### Container type
Add a container type for events (like an `Either` or `Validation`) so `Either = Left Error | Right val`.

### Existing function changes
```
// Map only the values of a stream to either values or errors.
flyd.map( (value) => value|error )

// Callback for only the values of a stream.
flyd.on( (*) => * )

// Combine streams into a dependent stream based only on the values from the streams being depended on.
flyd.combine( (v1, v2) => value|error, [s1, s2] )

// Merge both values and errors from two streams into one.
flyd.merge(s1, s2)
```

### New functions
```
// Map only the errors of a stream to an error or value.
flyd.mapErrors( (error) => value|error )

// Map events to any value, if they're events keep them, if they're values wrap them again.
flyd.mapAll( (event) => value|error|event )
```
