---
layout: doc
---

# Documentation

Unofficial documentation for the [coro-channel](https://github.com/luvit/lit/blob/master/deps/coro-channel.lua) module, version 3.0.2.

[coro-channel](https://github.com/luvit/lit/blob/master/deps/coro-channel.lua) is a wrapper module that provides manipulating read/write streams in a sync style of code using Lua coroutines without blocking the event loop.

----

## Functions

----

### wrapRead(socket) {#wrapRead}

Wraps a [stream handle](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) for a coroutine based reading.

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
local uv = require("uv") -- or luv outside of Luvi
local stdin = uv.new_tty(0, true) -- Create a readable stream, a TTY stream in this case
local reader, closer = wrapRead(stdin) -- Wrap the readable stream
for chunk in reader do -- Each time something is typed in console: execute reader and store the result in chunk
  if not chunk or chunk == '\n' then break end -- If the user inputs an empty chunk then stop reading
  print("You typed: " .. chunk) -- Output what the user have input
end
closer() -- Close the handle if reading has stopped
print("You have exited")
```

----

### wrapWrite(socket) {#wrapWrite}

Wraps a [stream](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) handle for a coroutine based writing.

#### Parameters {#wrapWrite-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| socket | [uv_stream_t](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) | The stream [handle](https://github.com/luvit/luv/blob/master/docs.md#uv_handle_t--base-handle) to be wrapped for writing. |

#### Returns {#wrapWrite-returns}

| Return | Type     | Description |
|:-------|:--------:|:------------|
| writer | function | See [writer](#writer). |
| closer | function | see [closer](#closer). |

#### Examples {#wrapWrite-examples}

- Timer writing to stdout TTY every second

```lua
local uv = require("uv") -- or luv outside of Luvi
local stdout = uv.new_tty(1, false) -- Create a writable stdout stream
local writer, closer = wrapWrite(stdout) -- Wrap the writable stream

local passed = 0 -- How many second have passed?
local function callback() -- A callback to be executed every second
  coroutine.wrap(function() -- A new coroutine each time
    passed = passed + 1
    writer(passed .. " seconds has passed!\n") -- Write to stdout
  end)()
end

writer("Hello to the timer! To exit press Ctrl + C\n")
local timer = uv.new_timer() -- Create a timer handle
timer:start(1000, 1000, callback) -- Start it and repeat every second
```

Note: In the previous example the callback could be defined in the following way:
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

Wraps a [stream](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) handle for both writing and reading.

Has the same effect to calling `wrapRead` and `wrapWrite` on a stream, such as:
```lua
local reader = wrapRead(stream)
local writer, closer = wrapWrite(stream)
```

#### Parameters {#wrapStream-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| socket | [uv_stream_t](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) | The stream [handle](https://github.com/luvit/luv/blob/master/docs.md#uv_handle_t--base-handle) to be wrapped for writing. |

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

#### Returns {#reader-returns}

| Return | Type     | Description | Provided On |
|:-------|:--------:|:------------|:-----------:|
| data   | string / boolean | Returns the chunk of data as string on success, otherwise false. | Always |
| error  | string   | A string containing the error code caused the failure. | Failure Only |

#### Examples {#reader-examples}

- Using a reader in an iterator

```lua
local handle = uv.new_tty(1, true) -- or any kind of streams
local reader = coro_channel.wrapRead(handle)

for chunk, err in reader do
  if err then
    print("An error has occurred: " .. err)
    break
  end
  print("Data chunk successfully read: " .. chunk)
end

print("Reading data done")
```

----

### writer(chunk) {#writer}

Yields the running coroutine and resumes it when done writing the provided chunk of data.

#### Parameters {#writer-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| chunk  | string / table / nil | `nil` indicates EOF and will completely close the stream if there's nothing to read, otherwise it will shutdown the writing duplex only.<br> If the provided value is a string it will be written to the stream as a single chunk.<br> If a table of strings is provided the strings will be written in a one system call as if they were concatenated. |

#### Returns {#writer-returns}

| Return | Type     | Description | Provided On |
|:-------|:--------:|:------------|:-----------:|
| success| boolean  | Whether or not the operation was successful. | Always |
| error  | string   | A string containing the error code caused this failure | Failure Only |

#### Examples {#writer-examples}

Assuming the following example of using `writer` with a TTY handle running in a coroutine:

```lua
local handle = uv.new_tty(0, false) -- or any kind of streams
local writer = coro_channel.wrapWrite(handle)

local tbl = {"Hello ", "There", "!!", "\n"}
local success, err = writer(tbl)
if not success then
  print("An error has occurred: " .. err)
  return
end
```

Note: usually though you don't use it like this with a TTY handle, it was used like this in the above example just to make it simple.

----


### closer() {#closer}

Closes the wrapped stream completely if it wasn't already closed. You cannot read or write from a closed stream. Note that this returns immediately, even if the stream isn't closed yet.

----
