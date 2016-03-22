## `flyd`








## `flyd.combine`

Create a new dependent stream

__Signature__: `(...Stream * -> Stream b -> b) -> [Stream *] -> Stream b`

### Parameters

* `fn` **`Function`** the function used to combine the streams
* `dependencies` **`Array<stream>`** the streams that this one depends on


### Examples

```js
var n1 = flyd.stream(0);
var n2 = flyd.stream(0);
var max = flyd.combine(function(n1, n2, self, changed) {
  return n1() > n2() ? n1() : n2();
}, [n1, n2]);
```

Returns `stream` the dependent stream


## `flyd.curryN`

Returns `fn` curried to `n`. Use this function to curry functions exposed by
modules for Flyd.

### Parameters

* `arity` **`Integer`** the function arity
* `fn` **`Function`** the function to curry


### Examples

```js
function add(x, y) { return x + y; };
var a = flyd.curryN(2, add);
a(2)(4) // => 6
```

Returns `Function` the curried function


## flyd.Either
Used to create `Right` and `Left` data types to put into streams. These data
types are wrappers around values. A `Right` value is meant to represent a
successful value while a `Left` represents a failure.

### Examples
```js
var s = flyd.stream(flyd.Either.Right(2));
s.right() // 2
```


## `flyd.endsOn`

Changes which `endsStream` should trigger the ending of `s`.

__Signature__: `Stream a -> Stream b -> Stream b`

### Parameters

* `endStream` **`stream`** the stream to trigger the ending
* `stream` **`stream`** the stream to be ended by the endStream
* `the` **`stream`** stream modified to be ended by endStream


### Examples

```js
var n = flyd.stream(1);
var killer = flyd.stream();
// `double` ends when `n` ends or when `killer` emits any value
var double = flyd.endsOn(flyd.merge(n.end, killer), flyd.combine(function(n) {
  return 2 * n();
}, [n]);
```



## `flyd.immediate`

Invokes the body (the function to calculate the value) of a dependent stream

By default the body of a dependent stream is only called when all the streams
upon which it depends has a value. `immediate` can circumvent this behaviour.
It immediately invokes the body of a dependent stream.

__Signature__: `Stream a -> Stream a`

### Parameters

* `stream` **`stream`** the dependent stream


### Examples

```js
var s = flyd.stream();
var hasItems = flyd.immediate(flyd.combine(function(s) {
  return s() !== undefined && s().length > 0;
}, [s]);
console.log(hasItems()); // logs `false`. Had `immediate` not been
                         // used `hasItems()` would've returned `undefined`
s([1]);
console.log(hasItems()); // logs `true`.
s([]);
console.log(hasItems()); // logs `false`.
```

Returns `stream` the same stream


## `flyd.isStream`

Returns `true` if the supplied argument is a Flyd stream and `false` otherwise.

__Signature__: `* -> Boolean`

### Parameters

* `value` **`Any`** the value to test


### Examples

```js
var s = flyd.stream(1);
var n = 1;
flyd.isStream(s); //=> true
flyd.isStream(n); //=> false
```

Returns `Boolean` `true` if is a Flyd streamn, `false` otherwise


## `flyd.map`

Map a stream

Returns a new stream consisting of all Right values and plain values from `s`
passed through `fn`. I.e. `map` creates a new stream that listens to `s` and
applies `fn` to every Right and plain value.
__Signature__: `(a -> result) -> Stream a -> Stream result`

### Parameters

* `fn` **`Function`** the function that produces the elements of the new stream
* `stream` **`stream`** the stream to map


### Examples

```js
var numbers = flyd.stream(0);
var squaredNumbers = flyd.map(function(n) { return n*n; }, numbers);
```

Returns `stream` a new stream with the mapped values


## `flyd.mapAll`

Like `flyd.map` except all values, including Lefts, are applied to the function.

__Signature__: `(a -> result) -> Stream a -> Stream result`

### Parameters

* `function` **`Function`** the function to apply


### Examples

```js
var numbers = flyd.stream(Either.Right(1));
var filtered = flyd.mapAll(function(v) {
  if (s.isRight()) return v.right;
}, s);
```

Returns `stream` a new stream with the values mapped


## `flyd.merge`

Creates a new stream down which all values from both `stream1` and `stream2`
will be sent.

__Signature__: `Stream a -> Stream a -> Stream a`

### Parameters

* `source1` **`stream`** one stream to be merged
* `source2` **`stream`** the other stream to be merged


### Examples

```js
var btn1Clicks = flyd.stream();
button1Elm.addEventListener(btn1Clicks);
var btn2Clicks = flyd.stream();
button2Elm.addEventListener(btn2Clicks);
var allClicks = flyd.merge(btn1Clicks, btn2Clicks);
```

Returns `stream` a stream with the values from both sources


## `flyd.on`

Listen to stream events

Similar to `map` except that the returned stream is empty. Use `on` for doing
side effects in reaction to stream changes. Use the returned stream only if you
need to manually end it. This ignores Lefts.

__Signature__: `(a -> result) -> Stream a -> Stream undefined`

### Parameters

* `cb` **`Function`** the callback
* `stream` **`stream`** the stream



Returns `stream` an empty stream (can be ended)


## `flyd.scan`

Creates a new stream with the results of calling the function on every incoming
stream with and accumulator and the incoming value.

__Signature__: `(a -> b -> a) -> a -> Stream b -> Stream a`

### Parameters

* `fn` **`Function`** the function to call
* `val` **`Any`** the initial value of the accumulator
* `stream` **`stream`** the stream source


### Examples

```js
var numbers = flyd.stream();
var sum = flyd.scan(function(sum, n) { return sum+n; }, 0, numbers);
numbers(2)(3)(5);
sum(); // 10
```

Returns `stream` the new stream


## `flyd.stream`

Creates a new stream

__Signature__: `a -> Stream a`

### Parameters

* `initialValue` **`Any`** (Optional) the initial value of the stream


### Examples

```js
var n = flyd.stream(1); // Stream with initial value `1`
var s = flyd.stream(); // Stream with no initial value
```

Returns `stream` the stream


## `flyd.transduce`

Creates a new stream resulting from applying `transducer` to `stream`.

__Signature__: `Transducer -> Stream a -> Stream b`

### Parameters

* `xform` **`Transducer`** the transducer transformation
* `source` **`stream`** the stream source


### Examples

```js
var t = require('transducers.js');

var results = [];
var s1 = flyd.stream();
var tx = t.compose(t.map(function(x) { return x * 2; }), t.dedupe());
var s2 = flyd.transduce(tx, s1);
flyd.combine(function(s2) { results.push(s2()); }, [s2]);
s1(1)(1)(2)(3)(3)(3)(4);
results; // => [2, 4, 6, 8]
```

Returns `stream` the new stream


## `stream.ap`

Returns a new stream which is the result of applying the
functions from `this` stream to the values in `stream` parameter.

`this` stream must be a stream of functions.

_Note:_ This function is included in order to support the fantasy land
specification.

__Signature__: Called bound to `Stream (a -> b)`: `a -> Stream b`

### Parameters

* `stream` **`stream`** the values stream


### Examples

```js
var add = flyd.curryN(2, function(x, y) { return x + y; });
var numbers1 = flyd.stream();
var numbers2 = flyd.stream();
var addToNumbers1 = flyd.map(add, numbers1);
var added = addToNumbers1.ap(numbers2);
```

Returns `stream` a new stram with the functions applied to values


## `stream.end`








## `stream.isLeft`

Returns `true` if the last value in the stream is a Left value.

__Signature__: Called bound to `Stream a`: `Boolean`

### Examples
```js
var s = flyd.stream(1);
s.isLeft(); // false
s.left(2);
s.isLeft(); // true
```


## `stream.isRight`

Returns `true` if the last value in the stream is a Right value or a plain
value.

__Signature__: Called bound to `Stream a`: `Boolean`

### Examples
```js
var s = flyd.stream(1);
s.isRight(); // true
s(Either.Right(2));
s.isRight(); // true
s.left(-1);
s.isRight(); // false
```


## `stream.left`

If an argument is applied, wraps a value in a Left and pushes it down the stream.

__Signature__: Called bound to `Stream a`: `a -> Stream (Left a)`

If no argument is applied, returns the last value of the stream if it is a left
value. If it is not, an error is thrown.

__Signature__: Called bound to `Stream a`: `a`

### Examples
```js
var s = flyd.stream();
s.left(1);
s.left(); // 1
s(Either.Left(2);
s.left(); // 2
s(3);
s.left(); // TypeError
```


## `stream.map`

Returns a new stream identical to the original except all Right values and plain
values will be passed through `f`. This ignores any Left values in the stream.

_Note:_ This function is included in order to support the fantasy land
specification.

__Signature__: Called bound to `Stream a`: `(a -> b) -> Stream b`

### Parameters

* `function` **`Function`** the function to apply


### Examples

```js
var numbers = flyd.stream(0);
var squaredNumbers = numbers.map(function(n) { return n*n; });
```

Returns `stream` a new stream with the values mapped


## `stream.mapAll`

Similar to `stream.map` except all values, including Lefts, are passed through
`f`.

__Signature__: Called bound to `Stream a`: `(a -> b) -> Stream b`

### Parameters

* `function` **`Function`** the function to apply


### Examples

```js
var numbers = flyd.stream(Either.Right(1));
var filtered = numbers.mapAll(function(v) { if (s.isRight()) return v.right; });
```

Returns `stream` a new stream with the values mapped


## `stream.of`



### Parameters

* `value` **`Any`** the initial value


### Examples

```js
var n = flyd.stream(1);
var m = n.of(1);
```

Returns `stream` the new stream


## `stream.right`

An alias for `stream`. Used to set the value of the stream or get the last Right
value out of the stream.


## `stream.toString`

Get a human readable view of a stream




Returns `String` the stream string representation
