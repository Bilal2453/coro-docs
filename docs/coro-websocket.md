---
layout: doc
---

# coro-websocket

Documentation for the [coro-websocket](https://github.com/luvit/lit/blob/master/deps/coro-websocket.lua) library, version 3.1.0.

[coro-websocket](https://github.com/luvit/lit/blob/master/deps/coro-websocket.lua) is a library that implements the [WebSocket](https://en.wikipedia.org/wiki/WebSocket) WS(s) protocol with a synchronous style interface making use of Lua coroutines.

### Installation

```sh
lit install luvit/coro-websocket
```

[On Lit search.](https://luvit.io/lit.html#author:luvit%20coro-websocket)

----

## Functions

----

### connect(options) {#connect}

Establishes a WebSocket connection with the said host.

***This method MUST be run in a coroutine***

#### Parameters {#connect-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| options | table | See [Options](#options) for details. |

#### Returns {#connect-returns}

| Name  | Type   | Description |
|:------|:------:|:------------|
| res   | [Response](#response) | The response of the WebSocket upgrade request. |
| read  | function | See [read](#read) for details. |
| write | function | See [write](#write) for details. |

#### Examples {#connect-examples}

- Writing and listening to an echo WSS server

```lua
-- Establish a new WSS connection to wss://echo.websocket.org:443
-- Which is an echo server, meaning, whatever you send will
-- get a response with the same contents you originally sent.
local res, read, write = connect {
  host = "echo.websocket.org",
  port = 443,
  tls = true
}

-- Did the connection get established successfully? if not what's the error?
if not res then
  print("Error establishing connection!\n" .. read); return
end

-- Send a message opcode 2 with the payload of "Hello There!"
write {
  payload = "Hello There!"
}

-- Execute the code inside the for loop each time we receive a message and then wait for more
for msg in read do
  -- Print the payload we got, since it is an echo server should be same as our request
  print("Host responded with " .. msg.payload)
  -- Listen to more stuff ...
  res.socket:shutdown() -- Stop listening and close the connection
end
print("Connection closed!")
```

----

### wrapIo(rawRead, rawWrite, options) {#wrapIo}

Wraps [coro-channel](https://bilal2453.github.io/coro-docs/docs/coro-channel.html) wrappers to be WS compatible. This assumes the raw wrapper can understand WebSocket protocol, meaning, it uses [WebSocket-Codec](https://github.com/luvit/lit/blob/master/deps/websocket-codec.lua) encoders&decoders adapters.

*This method does not require running in a coroutine*

#### Parameters {#wrapIo-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| rawRead | function | Supposedly [coro-channel reader](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader), or an [adapter](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html) of it that can correctly decode WebSocket frames. |
| rawWrite | function | Supposedly [coro-channel writer](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#writer), or an [adapter](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html#encoder) of it that can correctly encode WebSocket frames. |
| options | table | See [Options](#options) for details. |

#### Returns {#wrapIo-returns}

| Name  | Type   | Description |
|:------|:------:|:------------|
| read  | function | See [read](#read) for details. |
| write | function | See [write](#write) for details. |

----

### parseUrl(url) {#parseUrl}

Parses a WebSocket URL into a Lua table.

*This method does not require running in a coroutine*

#### Parameters {#parseUrl-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| url   | string | The WebSocket URL to be parsed, must be a valid WebSocket URL. |

#### Returns {#parseUrl-returns}

**On Success:**

| Name  | Type   | Description |
|:------|:------:|:------------|
| result| table  | The parsed URL as a table structure, see [WS URL](#ws-url) for details. |

**On Failure:**

| Name | Type   | Description |
|:-----|:------:|:------------|
| fail | nil    | If the first return was `nil`, that'd mean the operation has failed. |
| error| string | An error message explaining what went wrong. |

#### Examples {#parseUrl-examples}

```lua
local url = parseUrl("ws://ex.example.org/path/name")
print(string.format(
  "host: %s\nport: %d\npathname: %s\nWSS?: %s",
  url.host, url.port, url.pathname, url.tls and true or false
))
```

----

## Returned Functions

----

### write(message) {#write}

Sends (writes) a WebSocket [Message](#message) into the host stream while yielding the current coroutine, and resumes when done writing.

***This method MUST be run in a coroutine***

#### Parameters {#write-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| msg   | [Message](#message) | The WS(S) message to be written into the host stream. See [Message](#message) for more details. |

#### Returns {#write-returns}

| Name  | Type   | Description |
|:------|:------:|:------------|
| success | boolean | Whether or not the operation has succeeded. |
| error | string | A string containing the error code caused this failure. |

----

### read() {#read}

Yields the current coroutine till the stream to receive a WebSocket message, then resumes with the received message structure.

***This method MUST be run in a coroutine***  

#### Returns {#read-returns}

**On Success:**

| Name  | Type   | Description |
|:------|:------:|:------------|
| msg   | [Message](#message) | A received WebSocket message from the host. |

**On Failure:**

| Name  | Type   | Description |
|:------|:------:|:------------|
| success | boolean | A `false` value indicting the operation has failed. |
| error | string | A string containing the error code caused this failure. |

----

## Structures

### Message

String-indexed table that represents a WebSocket message, whether it's sent or received.

#### Fields {#message-fields}

| Field | Type   | Description |
|:-----:|:------:|:------------|
| fin   | boolean| Whether or not this is the last fragment in the message. See [RFC6455-Section 5.2](https://tools.ietf.org/html/rfc6455#section-5.2) for details. |
| rsv1  | boolean| The first reserved field, its meaning's defined by the host. See [RFC6455-Section 5.2](https://tools.ietf.org/html/rfc6455#section-5.2) for details. |
| rsv2  | boolean| The second reserved field, its meaning's defined by the host. See [RFC6455-Section 5.2](https://tools.ietf.org/html/rfc6455#section-5.2) for details. |
| rsv3  | boolean| The third reserved field, its meaning's defined by the host. See [RFC6455-Section 5.2](https://tools.ietf.org/html/rfc6455#section-5.2) for details. |
| opcode| number | Defines how the received payload should be interpreted. Those are defined by [RFC6455 Page-29](https://tools.ietf.org/html/rfc6455#page-29) and are handled by coro-websocket. |
| mask  | boolean| Whether or not the frame is bit masked. See [RFC6455-Section 5.1](https://tools.ietf.org/html/rfc6455#section-5.1) for more details.
| len   | number | The length(size) of the message payload in bytes. |
| payload | string | The actual contents of the message. |

----

### Options {#options}

A string-indexed table that provides further configurations over the ws connection and over how coro-websocket behave.

If an option field name matches one of coro-net [Options](https://bilal2453.github.io/coro-docs/docs/coro-net.html#options) name then same rules applies. All fields are only available to [connect](#connect), unless stated otherwise.

#### Fields {#options-fields}

| Field | Type   | Description |
|:-----:|:------:|:------------|
| path  | string | The pipe name the stream should refer to, only relevant if `host` and `port` aren't provided. |
| host  | string | The hostname the TCP socket should be referring to. If `port` is provided but `host` isn't, the default value is used. |
| port  | number | The port of the host the connection refers to. |
| tls   | boolean/table | Whether or not to use secure-layer on top of the connection. A table value for using secure-layer with further configurations, see [coro-net TLS Options](https://bilal2453.github.io/coro-docs/docs/coro-net.html#tls-options) for more details. |
| pathname | string | The path at which to connect to the host. That's, everything after the first `/`. |
| subprotocol | string | The WebSocket sub-protocol. See [RFC6455 Section 1.9](https://tools.ietf.org/html/rfc6455#section-1.9) for details. |
| headers | table | The outgoing request's headers. See [Headers](#headers) for details. |
| timeout | number | How many millisecond to wait before canceling the outgoing connection. |
| heartbeat | number | If set, automatically sends a heartbeat message (opcode 10) each `heartbeat` ms. Available to [wrapIo](#wrapIo) as well as [connect](#connect).|
| mask  | boolean | Whether or not to apply a mask on every message. Only available with [wrapIo](#wrapIo). |

### WS URL {#ws-url}

A table structure that represents a WebSocket URL.

#### Fields {#ws-url-fields}

| Field | Type   | Description |
|:-----:|:------:|:------------|
| host  | string | The hostname which this URL refer to. |
| port  | number | The port at which this URL open on the host. |
| tls   | boolean| Whether or not to use secure-layer, WSS instead of WS. |
| pathname | string | The path at which this URL refers to on the host. |

----

### Headers

An array of string list representing multiple headers, where the first value is the header name and the second is its value.

#### The Structuring {#headers-structuring}

{% raw %}

**Full**: `{{header-name, header-value}, ...}}`.

  **Where**:

| Entry       | Type   | Description             |
|:------------|:------:|:------------------------|
| header-name | string | The name of the header. |
| header-value| string | The value of the header.|

**Examples**:

- `{{"Connection", "Upgrade"}, {"Upgrade", "websocket"}}`.

{% endraw %}

----

### Response {#response}

A table structure that represents a successfully established TCP connection.

#### The Structuring {#response-structuring}

{% raw %}

**Full**:

```lua
  {
    header...,
    code = (number),
    reason = (string),
    socket = (uv_tcp_t),
    version = (number),
    keepAlive = (boolean),
  }
```

  **Where**:

| Entry        | Type   | Description                |
|:-------------|:------:|:---------------------------|
| header  | [header](#headers) | A sequence of [header](#headers) structures each individually. Meaning, each header is assigned to a numerical index in the structure.  |
| code         | number | The HTTP status code. 101 if WS connection has successfully upgraded. |
| reason       | string | The reason for getting the past status code (Reason-Phrase). |
| version      | number | The version of the used HTTP protocol as decimal number. |
| keepAlive    | boolean| Whether or not the connection should be kept alive. This would be `false` most often. |
| socket | [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle) | The TCP socket handle the WebSocket connection was bound to. |

{% endraw %}
