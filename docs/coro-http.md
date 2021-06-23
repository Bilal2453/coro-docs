---
layout: doc
---

# Documentation

Unofficial documentation for the library [coro-http](https://github.com/luvit/lit/blob/master/deps/coro-http.lua), version 3.2.0.

[coro-http](https://github.com/luvit/lit/blob/master/deps/coro-http.lua) is a library for manipulating the HTTP(s) protocol in a sync code-style making use of Lua coroutines, while actually keeping it async behind the scenes.

Can be used as a great replacement for the Luvit 2 built-in http & https libraries, to get rid of the callback code-style, with a more user-friendly interface. This is achieved through coroutines yielding and resuming without blocking the main event loop of [luv](https://github.com/luvit/luv/), although that means your requests must always run inside some kind of a coroutine, which some may consider as a downside since Luvit 2 doesn't wrap the main code chunk in a coroutine automatically.

Many thanks for [@trumedian](https://github.com/truemedian) for helping out with correcting many details, better wording, and pointing out typos.

----

## Functions

----

### request(method, url[, headers[, body[, options]]]) {#request}

Synchronously performs an HTTP(s) request after establishing a connection with the said host.

- This method will raise an error up upon failure. Advised to be called with pcall/xpcall.

- If said connection \[with the host specified in `url`\] was already established and is still alive, it will be used instead of establishing another connection. Generally this is done either by a previous `request` call or by manually calling `saveConnection`.

***This method MUST be run in a coroutine***

#### Parameters {#request-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| method| string | An all uppercase HTTP method name. | ❌ |
| url   | string | An HTTP URL to send the request to. | ❌ |
|headers| [http-header](#http-header) | The HTTP headers of the request. | ✔ |
| body  | string | The request's payload (if needed). | ✔ |
| options | [Request-Options](#request-options)/[Timeout](#timeout) | - If a number is supplied, this will act as the timeout to wait before giving up on receiving a response.<br>- If a table is supplied, this will act as a table to provide additional configurations. See [Request-Options](#request-options) for more details.  | ✔ |

#### Returns {#request-parameters}

| Name | Type   | Description |
|:-----|:------:|:------------|
| res  | [Response](#request-response) | A [Response](#request-response) structure representing the received response. |
| body | string | The response payload (body) the server responded with as a string if any, otherwise an empty string. |

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

#### onConnect {#createServer-onConnect}

A callback that will asynchronously be called each time a new connection is established to the server. That is, each time a client is connecting to the created server.

- You are suppose to return at least one value each time this callback is executed, as in: `head, body`. Where `head` is a [Response](#request-response) structure, and `body` is optionally a string representing the response payload.

- Be aware that coro-http __WILL NOT__ auto-set any headers, meaning if you are going to provide a `body`, make sure you set `Content-Length` header as well.

The callback has the following parameters:

| Param | Type   | Description |
|:------|:------:|:------------|
| req   | [Request](#request-response) | The request's headers and general information. |
| body  | string | The provided request's payload as a string, empty string incase no payload is provided. |
| socket| [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle)/[uv_pipe_t](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle)| The socket that the connection was bound to. |

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

----

### getConnection(host, port [, tls [, timeout]]) {#getConnection}

Establishes a new TCP connection with the given host on the given port.

- If the connection was previously saved using `saveConnection`, calling this will return that connection and un-save it.

- If the saved connection was closed, or there were none previously saved, a new connection will be established instead.

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

*This method does not require running in a coroutine*

#### Parameters {#saveConnection-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| connection  | [TCP Connection](#tcp-connection) | A table representing a TCP connection returned by `getConnection`. |

----

## Structures

Here are the data structures (tables usually) used by the library's methods, either for a returned valueor for a parameter.

### HTTP Header

A table structure representing an HTTP(s) header. The structure is an array of two strings, the first entry is the header-name, and the second entry is the header-value. See the [rfc2616 paper](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html) for more details about the officially acceptable HTTP headers.

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

Represents an HTTP(s) request or a response including the headers, and general information about the connection. The exact data is highly dependent on the server/client and therefore the docs cannot tell exactly what fields you are going to receive. If you want to know such information, you should either debug your code, or read the API manual of the said server.

#### Structure {#request-response-structure}

**Full**:

```lua
  {
    http_header...,
    code = (number),
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

A table structure that represents a TCP connection (wrapped using coro-channel). The structure offers methods to directly read and write from the socket, and other details about the connection. Generally speaking you should only use these when it is the only way to accomplish what you need since they are a bit for advanced users.

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

### Request Options {#request-options}

A table that could configure some of [Request](#request) behavior

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
