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

@todo: API with errors in mind
