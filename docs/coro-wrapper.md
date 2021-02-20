# Documentations

Unofficial docs for the [coro-wrapper](https://github.com/luvit/lit/blob/master/deps/coro-wrapper.lua) module, version 3.1.0.

[coro-wrapper](https://github.com/luvit/lit/blob/master/deps/coro-wrapper.lua) is an adapter module for applying decoders to [coro-channel](https://bilal2453.github.io/coro-docs/docs/coro-channel.html).

----

## Functions

----

### merger(read, scan)

Returns an adapter that merges multiple chunks together as a one return per call.

#### Parameters

| Param | Type     | Description |
|:-----:|:--------:|:------------|
| read  | function | Supposedly the coro-channel's [reader](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader) you'll be adapting, though any wrapper that returns a chunk of data each time it is called would work. |
| scan  | function | A callback that takes the read chunk of data, and either returns true to flush and return the buffer, or a falsely value to keep waiting for more data to be read. |

#### Returns

| Return | Type     | Description |
|:------:|:--------:|:------------|
| read   | function | Reads stream chunks (possibly while yielding the current coroutine) till the whole buffer is consumed or till `scan` returns a truthy value, then returns the read chunks concatenated in a single string buffer. |
| updateScan | function | A function that takes a single function parameter and set `scan` to that input. |

----

### decoder(read, decode)

Returns an adapter of `read`, where it keeps building a buffer of the read chunks, and when enough data is supplied, it will be decoded using `decode` and the return of that will be the adapter result, while still allowing you to hot-swipe the decoder.

#### Parameters

| Param | Type     | Description |
|:-----:|:--------:|:------------|
| read  | function | Supposedly the coro-channel's [reader](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader) you'll be adapting, though any wrapper that returns a chunk of data each time it is called would work. |
| decode| function | A callback that takes the string buffer and its current index and gets called on each chunk read.<br> If a `nil` is returned the adapter will keep listening for more data to build the buffer with.<br>Otherwise, the returned value will be considered as the decoder result,meaning that value will be returned by the adapter too, and the buffer will be flushed.<br> If a second value of type number is returned that would be considered as the new buffer index to start at, meaning, any data in the buffer before the supplied index will be trimmed down. |

#### Returns

| Return | Type     | Description |
|:------:|:--------:|:------------|
| read   | function | Reads stream chunks (possibly while yielding the current coroutine) till enough data is supplied (determined by `decode`), then returns the decoded item by `decode`. |
| updateDecode | function | A function that takes a single function parameter and set `decode` to that input. |

----

### encoder(write, encode)
 
Returns an adapter of `write`, where it allows you to write the returns of `encode` into the stream while at the same time allowing you to hot-swipe the encoder.

#### Parameters

| Param | Type     | Description |
|:-----:|:--------:|:------------|
| write | function | Supposedly the coro-channel's [writer](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#writer) you'll be adapting, though any wrapper that takes input and writes it to stream would work. |
| encode| function | A callback that takes the adapter input and returns the actual data chunk to be written. |

#### Returns

| Return | Type   | Description |
|:------:|:------:|:------------|
| ...    | Any    | Returns everything `write` returns. |

----
