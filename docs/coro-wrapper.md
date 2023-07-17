---
layout: doc
---

# coro-wrapper

Documentation for the [coro-wrapper](https://github.com/luvit/lit/blob/master/deps/coro-wrapper.lua) module, version 3.1.0.

[coro-wrapper](https://github.com/luvit/lit/blob/master/deps/coro-wrapper.lua) is an adapter module for applying decoders/encoders/merger adapters to [coro-channel](https://bilal2453.github.io/coro-docs/docs/coro-channel.html) readers and writers.

Understanding this and how it works can be a bit tricky.  But think about it like this:  You have a reader that returns an HTTP chunk back as the raw HTTP string, and you want to make that reader return a parsed version of that HTTP chunk, so you apply the `decoder` adapter on that reader to provide an abstraction over the lower level return, this is in fact how [coro-http](https://bilal2453.github.io/coro-docs/docs/coro-http.html) does it!
The rest of the adapters has a similar concept.

### Installation

```sh
lit install creationix/coro-wrapper
```
[On Lit search.](https://luvit.io/lit.html#coro-wrapper)

----

## Functions

----

### merger(read, scan) {#merger}

Returns an adapter that merges multiple chunks together as a one return per call.

#### Parameters {#merger-parameters}

| Param | Type     | Description |
|:-----:|:--------:|:------------|
| read  | function | Supposedly the coro-channel's [reader](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader) you'll be adapting, though any wrapper that returns a chunk of data each time it is called would work. |
| scan  | function | A callback that takes the read chunk of data, and either returns true to flush and return the buffer, or a falsely value to keep waiting for more data to be read. |

#### Returns {#merger-returns}

| Return | Type     | Description |
|:------:|:--------:|:------------|
| read   | function | Reads stream chunks (possibly while yielding the current coroutine) till the whole buffer is consumed or till `scan` returns a truthy value, then returns the read chunks concatenated in a single string buffer. |
| updateScan | function | A function that takes a single function parameter and set `scan` to that input. |

----

### decoder(read, decode) {#decoder}

Returns an adapter of `read`, where it keeps building a buffer of the read chunks, and when enough data is supplied, it will be decoded using `decode` and the return of that will be the adapter result, while still allowing you to hot-swipe the decoder.

#### Parameters {#decoder-parameters}

| Param | Type     | Description |
|:-----:|:--------:|:------------|
| read  | function | Supposedly the coro-channel's [reader](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader) you'll be adapting, though any wrapper that returns a chunk of data each time it is called would work. |
| decode| function | A callback that gets called on each chunk read while taking the string buffer and its current index as inputs.<br> If a `nil` is returned the adapter will keep listening for more data to build the buffer with.<br>Otherwise, the returned value will be considered as if it were the decoder result, and the buffer will be flushed.<br> If a second value of type number is returned that would be considered as the new buffer index to start at, meaning, any data in the buffer before the supplied index will be trimmed down. |

#### Returns {#decoder-returns}

| Return | Type     | Description |
|:------:|:--------:|:------------|
| read   | function | Reads stream chunks (possibly while yielding the current coroutine) till enough data is supplied (determined by `decode`), then returns the decoded item by `decode`. |
| updateDecode | function | A function that takes a single function parameter and set `decode` to that input. |

----

### encoder(write, encode) {#encoder}
 
Returns an adapter of `write`, where it allows you to write the returns of `encode` into the stream while at the same time allowing you to hot-swipe the encoder.

#### Parameters {#encoder-parameters}

| Param | Type     | Description |
|:-----:|:--------:|:------------|
| write | function | Supposedly the coro-channel's [writer](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#writer) you'll be adapting, though any wrapper that takes input and writes it to stream would work. |
| encode| function | A callback that takes the adapter input and returns the actual data chunk to be written. |

#### Returns {#encoder-returns}

| Return | Type   | Description |
|:------:|:------:|:------------|
| ...    | Any    | Returns everything `write` returns. |

----
