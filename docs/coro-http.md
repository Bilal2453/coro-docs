---
layout: doc
---

# Documentation

Documentation for the library [coro-http](https://github.com/luvit/lit/blob/master/deps/coro-http.lua), version 3.2.3.

[coro-http](https://github.com/luvit/lit/blob/master/deps/coro-http.lua) is a library implementing an HTTP 1.1 client and server. It uses a sync code-style making use of Lua coroutines, while keeping everything asynchronous behind the scenes. That is, it yields but it does NOT block any threads.

Can be used as a great replacement for the Luvit built-in http & https libraries, to get rid of the callback code-style, with a more user-friendly interface. This is achieved through coroutines yielding and resuming without blocking the main event loop of [luv](https://github.com/luvit/luv/).

Keep in mind, Luvit (starting with [version 2.18](https://github.com/luvit/luvit/pull/1139)) already wraps the main file in a coroutine for you!

Many thanks for [@trumedian](https://github.com/truemedian) for helping out with this!

----

## Functions

----

### request(method, url[, headers[, body[, options]]]) {#request}

Synchronously performs a non-blocking HTTP(s) request after establishing a connection with the said host.

- This method will raise an error on network failure. Advised to be called with `pcall`/`xpcall`.

- If the needed connection \[with the host specified in `url`\] has been previously established and is still alive, it will be reused instead of establishing a new one. Generally this happen by a previous `request` call or by manually calling `saveConnection`.

- Be aware that coro-http will try to auto-set some headers if the user doesn't supply them, specifically `Content-Length` and `Host`.  `Host` will be set to the hostname if not provided.  `Content-Length` is set to the length of `body` when the request is not chunked (`Content-Encoding` is not `chunked`) and it wasn't already provided by the user.

***This method MUST be run in a coroutine***

#### Parameters {#request-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| method| string | An all uppercase HTTP method name. | ❌ |
| url   | string | An HTTP(s) URL to send the request to. | ❌ |
|headers| [http-header](#http-header) | The HTTP headers of the request. | ✔ |
| body  | string | The request's payload (if needed). | ✔ |
| options | [Request-Options](#request-options)/[Timeout](#timeout) | - If a number is supplied, this will act as the timeout to wait before giving up on receiving a response.<br>- If a table is supplied, this will act as a table to provide additional configurations. See [Request-Options](#request-options) for more details.  | ✔ |

#### Returns {#request-returns}

| Name | Type   | Description |
|:-----|:------:|:------------|
| res  | [Response](#request-response) | A [Response](#request-response) structure representing the received response. |
| body | string | The response payload (body) the server responded with as a string if any, otherwise an empty string. |

#### Examples {#request-examples}

- Requesting Google's main page

```lua
local res, body = request("GET", "https://www.google.com")
if res.code ~= 200 then
   print("Could not fetch www.google.com successfully: " .. res.reason); return
end
print("Received Google main page HTML: " .. body)
```

- Sending a Discord webhook using POST request

```lua
local json = require("json")

local webhook_url = "https://discord.com/api/webhooks/{ID}/{TOKEN}" -- Your webhook URL here

-- Discord expects a body that is a JSON string
-- we use the built-in json module to encode a Lua table into JSON
local body = json.encode{
   content = "Hello There!\nThis is an example for a POST request using coro-http!"
}

local headers = {
   {"Content-Type", "application/json"}, -- we are sending a JSON form string for the body
   {"User-Agent", "MyWebhookClient"}. -- an optional example UA
}

-- Send the POST request
local res = request("POST", webhook_url, headers, body, 5000)

-- Was the request successful?
if res.code < 200 or res.code >= 300 then
   return print("Failed to send webhook: " .. res.reason)
end
print("Webhook sent successfully!")
```

----

### createServer(host, port, onConnect) {#createServer}

Creates a new server instance and asynchronously binds it to host:port.

*This method does not require running in a coroutine*

#### Parameters {#createServer-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| host  | string | The host which the server corresponds to. |
| port  | number | The port to which the created server should listen on. |
| onConnect | function | See [onConnect](#createServer-onConnect) for details. |

#### Returns {#createServer-returns}

| Name | Type   | Description |
|:-----|:------:|:------------|
| server | [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle) | The TCP socket of the created server. Server connection can be stopped and manipulated using this. |

#### onConnect {#createServer-onConnect}

A callback that will be asynchronously called each time a new connection is established to the server. That is, when a connection is received.

- For a response the callback **must** return at least one value, and at most two values, those returns are: `head, body`. Where `head` is a [Response](#request-response) structure, and `body` is an optional string payload.

- If a second value was returned (for the payload), a `Content-Length` header must be set to the length of the returned payload. Chunked responses **are not** supported.

The callback has the following parameters:

| Param | Type   | Description |
|:------|:------:|:------------|
| req   | [Request](#request-response) | The request's headers and general information. |
| body  | string | The provided request's payload as a string, empty string incase no payload is provided. |
| socket| [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle)/[uv_pipe_t](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle) | The socket that the connection was bound to. |

#### Examples {#createServer-examples}

- A local server that responds with 200

```lua
local res_payload = "Hello!"
local res_headers = {
   {"Content-Length", tostring(#res_payload)}, -- Must always be set if a payload is returned
   {"Content-Type", "text/plain"}, -- Type of the response's payload (res_payload)
   {"Connection", "close"}, -- Whether to keep the connection alive, or close it
   code = 200,
   reason = "OK",
}

-- 127.0.0.1 is localhost
createServer("127.0.0.1", 8080, function(req, body)
   print(req.method .. " request with the payload of '" .. body .. "'")
   return res_headers, res_payload -- respond with this to every request
end)
```

----

### parseUrl(url) {#parseUrl}

Parses the given string representing a valid HTTP(s) URL into a Lua table.

*This method does not require running in a coroutine*

#### Parameters {#parseUrl-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| url   | string | The HTTP URL to be parsed. An error will be raised up if it is not a valid HTTP URL. |

#### Returns {#parseUrl-returns}

| Name | Type    | Description |
|:-----|:-------:|:------------|
| result | table | A [Parsed URL](#parsed-url) structure representing the URL as a Lua table. |

#### Examples {#parseUrl-examples}

- Parse a url and print its information

```lua
local url = "https://www.example:8080.com/path/to/index"
local pu = parseUrl(url)
print(([[
hostname: %s
path: %s
host: %s
port: %d
HTTPS?: %s
]]):format(
   pu.hostname, pu.path, pu.host, pu.port,
   pu.tls and true or false
))
```

----

### getConnection(host, port [, tls [, timeout]]) {#getConnection}

Establishes a new TCP connection with the given host on the given port.

- If the connection was previously saved using `saveConnection`, calling this will return that connection and un-save it.

- If the saved connection was closed, or there were none previously saved, a new connection will be established instead.

- User should never need this in normal scenarios.

***This method MUST be run in a coroutine***

#### Parameters {#getConnection-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| host  | string | The host which the established connection refers to. | ❌ |
| port  | number | The port that this connection should refer to when connecting to the host. | ❌ |
| tls   | boolean/[TLS Options](#tls-options) | The use of SSL/TLS cryptographic protocol. <br> - Boolean value whether to use SSL/TLS cert or not.<br>- Table value to use SSL/TLS, with optional configurations. See [TLS Options](#tls-options) for acceptable fields.| ✔ <br> Default: `false`. |
| timeout   | [Timeout](#timeout) | How much time to wait for the response before canceling the request out. | ✔ |

#### Returns {#getConnection-returns}

| Name | Type   | Description |
|:-----|:------:|:------------|
| connection | [TCP Connection](#tcp-connection) | The established connection. |

----

### saveConnection(connection) {#saveConnection}

Saves a pre-established [TCP connection](#tcp-connection) to be used later instead of establishing a new one (e.g. would be used by [request](#request)).

- If the passed connection is dead nothing will be saved.

- User should never need this in normal scenarios.

*This method does not require running in a coroutine*

#### Parameters {#saveConnection-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| connection  | [TCP Connection](#tcp-connection) | A table representing a TCP connection returned by `getConnection`. |

----

## Structures

Here are the data structures (tables mostly) used by the library's methods, either as returns or as arguments.

Whenever one of those structures is used throughout the documentation, it will be linked to here for further details.

Each structure has a description, and possibly a `Structure` section that describes the table value.  The syntax `{foo, bar}` represents an array that has exactly two elements, `tbl[1]` and `tbl[2]`, what those values exactly are will be described below it in the `Where` section.  The syntax `{foo...}` represents an array that could contain any size of element `foo`.  The syntax `{foo = (type)}` represents a dictionary table that has a field `foo` which is of type `(type)`.  Table values could contain multiple of those syntaxes.

### HTTP Header

A table structure representing an HTTP(s) header. The structure is an array of two strings, the first entry is the header-name, and the second entry is the header-value. See [RFC2616 specification Section 14](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html) for more details about the officially acceptable HTTP 1.1 headers.

#### Structure {#http-header-structure}

**Full**: `{header-name, header-value}`.
  
- **Where**:

| Entry       | Type   | Description             |
|:------------|:------:|:------------------------|
| header-name | string | The name of the header. |
| header-value| string | The value of the header.|

**Examples**:

- `{"Expires", "-1"}`.
- `{"Accept", "text/plain"}`.

----

### Request/Response {#request-response}

Represents an HTTP(s) request or a response including the headers, and other general information about the connection. The exact data is highly dependent on the server/client and therefore the docs cannot tell exactly what fields to expect. If you need to know the exact provided fields, you should either debug your code or read the API manual of the said server.

#### Structure {#request-response-structure}

**Full**:

```lua
  {
    http_header...,
    code = (number),
    method = (string),
    reason = (string),
    version = (number),
    keepAlive = (boolean),
  }
```

- **Where**:

| Entry        | Type   | Description                |
|:-------------|:------:|:---------------------------|
| http_header  | [http-header](#http-header) | A sequence of [http-header](#http-header) structures each individually. Meaning, each header is assigned to a numerical index in the structure.  |
| code         | number | The HTTP status code. |
| reason       | string | The reason for getting the past status code (Reason-Phrase). |
| version      | number | The version of the used HTTP protocol as decimal number. |
| keepAlive    | boolean| Whether or not the connection should be kept alive. |

**Examples**:

{% raw %}

```lua
{
   {"Content-Type", "text/html"},
   {"Content-Length", "1587"},
   code = 200,
   reason = "OK",
   version = 1.1,
   keepAlive = true
}
```

{% endraw %}

----

### TCP Connection {#tcp-connection}

A table structure that represents a TCP connection (wrapped using coro-channel). The structure offers methods to directly read and write from the socket, and other details about the connection. Generally speaking you should only use these when it is the only way to accomplish what you need since they are for advanced uses.

#### Available Fields {#tcp-connection-fields}

| Field | Type   | Description |
|:------|:------:|:------------|
| socket| [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle) | The TCP socket (handle) used to bind the established connection. |
| host | string | Same as the passed parameter `host` when establishing. |
| port | number | Same as the passed parameter `port` when establishing. |
| tls  | table/boolean | Same as the passed parameter `tls` when establishing. |
| read | function | A [coro-channel reader](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader) that receives a new chunk of data each time it is resumed. |
| write| function | A [coro-channel writer](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#writer) that sends a chunk of data each time it is called through the socket. |
| updateEncoder | function | Takes a single function argument, and uses the said argument as the new [encoder](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html#encoder). |
| updateDecoder | function | Takes a single function argument, and uses the said argument as the new [decoder](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html#decoder). |
| reset | function | Updates the decoder back to the original, `httpCodec.decoder()`. |

----

### TLS Options {#tls-options}

Here are the available options and fields for configuring an SSL/TLS connection.

#### Acceptable Fields {#tls-options-fields}

| Field | Type   | Description |
|:------|:------:|:------------|
| protocol | string | The transport-layer-secure protocol to use. Supported values depends on lua-openssl version, openssl TLS 1.3 compatible supports: `TLS` (default) or `DTLS`. LibreSSL and others uses `SSLv23` by default and supports other versions. |
| ciphers | string | The encryption algorithm to encrypt data with, value **MUST** be a valid cipher suite string. Defaults are `TLS_AES_128_GCM_SHA256:TLS_AES_128_CCM_SHA256` for TLS 1.3, `ECDHE-RSA-AES128-SHA256:AES128-GCM-SHA256` for LTS 1.2, `RC4:HIGH:!MD5:!aNULL:!EDH` for LTS 1.0. |
| key | string | The PEM key of the supplied certification (if `cert` field is passed). |
| cert | string | The SSL/TLS x509 certification used for the handshake as string. Used alongside with field `key` or it gets ignored. |
| ca | string/table | The x509 root certificates to check the connection against. Defaults to a root certification (`root_ca.dat` file) when available. |
| insecure | boolean | Weather or not to accept invalid certificates on handshakes. Please only tinker with this when you do know what it means. ***default***: `false`. |

*All of the fields are optional and should only be touched when you know what you are dealing with.*

----

### Parsed URL {#parsed-url}

A parsed URL is often returned by [parseUrl](#parseUrl) as a table that represents an HTTP(s) URL.

#### Available Fields {#parsed-url-fields}

| Fields   | Type   | Description |
|:-------- |:------:|:------------|
| tls      | boolean| Whether or not the parsed URL uses TLS/SSL (HTTPS). |
| host     | string | The authority of the parsed URL (hostname:port). |
| hostname | string | The host name of the parsed URL (hostname only). |
| port     | number | The host port of the parsed URL if supplied(defaults to `80` for HTTP and `443` for HTTPS). |
| path     | string | Everything following the host of the parsed URL (including first `/`) |

----

### Timeout {#timeout}

A numerical value in milliseconds that indicates how much time to wait for the response/request before canceling it out. If nothing is supplied [libuv](https://github.com/libuv/libuv) will timeout after an undefined amount of seconds.

**Examples**:

- `1000`, waits for a one second.
- `500`, waits for half of a second.

----

### Request Options {#request-options}

A table that could configure some of [Request](#request) behavior.

#### Structure {#request-options-structure}

**Full**:

```lua
   {
      timeout = (number),
      followRedirects = (boolean)
   }
```

- **Where**:

| Entry          | Type   | Description |
|:---------------|:------:|:------------|
| timeout        | [Timeout](#timeout) | See [Timeout](#timeout) for description. |
| followRedirects| boolean | whether or not to follow redirect requests. ***default***: `true`. |

----
