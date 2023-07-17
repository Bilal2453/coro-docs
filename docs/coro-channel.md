---
layout: doc
---

# coro-channel

Documentation for the [coro-channel](https://github.com/luvit/lit/blob/master/deps/coro-channel.lua) module, version 3.0.3.

[coro-channel](https://github.com/luvit/lit/blob/master/deps/coro-channel.lua) is a wrapper module that wraps [stream handles](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) (or any other handle that inherits stream) to provide a sync style read/write interface making use of Lua coroutines, and without blocking the event loop.

### Installation

```sh
lit install creationix/coro-channel
```
[On Lit search.](https://luvit.io/lit.html#coro-channel)

----

## Functions

----

### wrapRead(socket) {#wrapRead}

Wraps a [stream handle](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) for coroutine based reading.

*This method does not require running in a coroutine*

#### Parameters {#wrapRead-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| socket | [uv_stream_t](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) | The stream [handle](https://github.com/luvit/luv/blob/master/docs.md#uv_handle_t--base-handle) to be wrapped for reading. |

#### Returns {#wrapRead-returns}

| Return | Type     | Description |
|:-------|:--------:|:------------|
| reader | function | See [reader](#reader). |
| closer | function | see [closer](#closer). |

#### Examples {#wrapRead-examples}

- A standard input echo example<br>(Type something in the console to get an echo)

```lua
local uv = require("uv") -- or "luv" outside of Luvi
local wrapRead = require("coro-channel").wrapRead

local stdin = uv.new_tty(0, true) -- create a readable stream, a TTY stream in this case
local reader, closer = wrapRead(stdin) -- wrap the readable stream
for chunk in reader do -- each time something is typed in console: execute reader and store the result in chunk
  if chunk == '\n' then break end -- if the input is a new line (Enter) stop reading 
  print("You typed: " .. chunk) -- output the user input
end
closer() -- close the handle if reading has stopped
print("You have exited")
```

Note: In the above example we won't receive an End of Stream (EoS), so we break manually on new lines, but this is not always the case, for example the coro-net reader will return `nil` when an EoS is received which would automatically break out of the for loop.

----

### wrapWrite(socket) {#wrapWrite}

Wraps a [stream handle](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) for coroutine based writing.

#### Parameters {#wrapWrite-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| socket | [uv_stream_t](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) | The [stream handle](https://github.com/luvit/luv/blob/master/docs.md#uv_handle_t--base-handle) to be wrapped for writing. |

#### Returns {#wrapWrite-returns}

| Return | Type     | Description |
|:-------|:--------:|:------------|
| writer | function | See [writer](#writer). |
| closer | function | see [closer](#closer). |

#### Examples {#wrapWrite-examples}

- Timer writing to stdout TTY every second

```lua
local uv = require("uv") -- or "luv" outside of Luvi
local wrapWrite = require("coro-channel").wrapWrite

local stdout = uv.new_tty(1, false) -- create a writable stdout stream
local writer, closer = wrapWrite(stdout) -- wrap the writable stream

local passed = 0 -- how many second have passed?
local function callback() -- a callback to be executed every second
  coroutine.wrap(function() -- a new coroutine each time
    passed = passed + 1
    writer(passed .. " seconds has passed!\n") -- write to stdout
  end)()
end

writer("Hello to the timer! To exit press Ctrl + C\n")
local timer = uv.new_timer() -- create a timer handle
timer:start(1000, 1000, callback) -- start it and repeat every second
```

Note: In the previous example the callback could be defined in the following way to avoid creating a new one everytime:
```lua
local callback = coroutine.wrap(function()
  while true do
    passed = passed + 1
    writer(passed .. " seconds has passed!\n")
    coroutine.yield()
  end
end)
```

----

### wrapSteam(socket) {#wrapStream}

Wraps a [stream handle](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) for both writing and reading.

Has the same effect to calling `wrapRead` and `wrapWrite` on the handle, such as:
```lua
local reader = wrapRead(stream)
local writer, closer = wrapWrite(stream)
```

#### Parameters {#wrapStream-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| socket | [uv_stream_t](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) | The [stream handle](https://github.com/luvit/luv/blob/master/docs.md#uv_handle_t--base-handle) to be wrapped for reading and writing. |

#### Returns {#wrapStream-returns}

| Return | Type     | Description |
|:-------|:--------:|:------------|
| reader | function | See [reader](#reader). |
| writer | function | See [writer](#writer). |
| closer | function | see [closer](#closer). |

----

## Returned Functions

----

### reader() {#reader}

Yields the running coroutine and resumes it after receiving a chunk of data.
Effectively: wait until there is data to read, then return the read data.

#### Returns {#reader-returns}

| Return | Type     | Description | Provided On |
|:-------|:--------:|:------------|:-----------:|
| data   | string / boolean | Returns the chunk of data as string on success, otherwise false. | Always |
| error  | string   | A string containing the error code caused the failure. | Failure Only |

#### Examples {#reader-examples}

- Using a reader in an iterator

```lua                             
local uv = require("uv") -- or "luv" outside of luvi
local wrapRead = require("coro-channel").wrapRead

local handle = uv.new_tty(1, true) -- or any kind of streams
local reader = wrapRead(handle)

for chunk, err in reader do
  if err then
    print("An error has occurred: " .. err)
    break
  elseif chunk == '\n' then -- press Enter to exit the tty
    break
  end
  print("Data chunk successfully read: " .. chunk)
end

print("Reading data done")
```

----

### writer(chunk) {#writer}

Yields the running coroutine and resumes it when done writing the provided chunk of data.
Effectively: write the data and wait until it has been written, then return.

#### Parameters {#writer-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| chunk  | string / table / nil | `nil` indicates End of Stream and will completely close the stream if there's nothing to read, if there is, it will only shutdown the writing duplex.<br> If the provided value is a string it will be written to the stream as a single chunk.<br> If a table of strings is provided the string values will be concatenated and written in a single system call. |

#### Returns {#writer-returns}

| Return | Type     | Description | Provided On |
|:-------|:--------:|:------------|:-----------:|
| success| boolean  | Whether or not the operation was successful. | Always |
| error  | string   | A string containing the error code caused this failure | Failure Only |

#### Examples {#writer-examples}

Assuming the following example of using `writer` with a TTY handle running in a coroutine:

```lua
local uv = require("uv") -- or "luv" outside of Luvi
local wrapWrite = require("coro-channel").wrapWrite

local handle = uv.new_tty(0, false) -- or any kind of streams
local writer = wrapWrite(handle)

local tbl = {"Hello ", "There", "!!", "\n"}
local success, err = writer(tbl)
if not success then
  print("An error has occurred: " .. err)
  return
end
```

Note: usually though this example is not how you use it with a TTY handle, the example is simplified and for explanation purposes only.

----


### closer() {#closer}

Closes the wrapped stream handle if it hasn't been already closed.
You cannot read or write from/to a closed stream.

Note: This call returns immediately, even if the stream hasn't fully closed yet.
