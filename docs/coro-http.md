---
layout: doc
---

# Documentation

Unofficial docs for the [coro-http](https://github.com/luvit/lit/blob/master/deps/coro-http.lua) module, version 3.2.0.

[coro-http](https://github.com/luvit/lit/blob/master/deps/coro-http.lua) is a library for manipulating HTTP(s) requests in a sync styled code using Lua coroutines.

It can be used as a great replacement for the Luvit built-in http library, to get rid of the callback style, and to have more user friendly interface. This is achieved through the use of coroutines to keep the sync style of your code without blocking the main event loop of [luv](https://github.com/luvit/luv/).

Many thanks for [@trumedian](https://github.com/truemedian) for helping out with correcting many invalid details, better wording and terms, and pointing out typos.

----

## Functions

----

### createServer(host, port, onConnect) {#createServer}

Creates a new server instance and asynchronously binds it to host:port.

*This method does not require running in a coroutine*

#### Parameters {#createServer-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| host  | string | The host which the server corresponds to.
| port  | number | The port to which the created server should listen on.
| onConnect | function | See [callback](#createServer-callback) for details. |

#### Callback {#createServer-callback}

A callback that will be asynchronously called every time a new connection is established to the server

The callback has the following parameters:

| Param | Type   | Description |
|:------|:------:|:------------|
| req   | [Request](#request-response) | The request headers and status. |
| body  | string | The provided request's payload as string, empty string in case nothing is provided. |
| socket| [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle) / [uv_pipe_t](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle)| The socket that the connection was bound to. |

You are suppose to return at least one value every time this callback is called: `head, body` where `head` is a [Response](#request-response) structure, and `body` is optionally a string representing the response payload.

----

### parseUrl(url) {#parseUrl}

Parses the given string representing a valid HTTP(s) URL into a Lua table.

*This method does not require running in a coroutine*

#### Parameters {#parseUrl-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| url   | string | The HTTP URL to be parsed. An error will be raised up of it isn't a valid URL. |

#### Returns {#parseUrl-returns}

| Name | Type    | Description |
|:-----|:-------:|:------------|
| result | table | A [Parsed URL](#parsed-url) structure representing the URL. |

----

### getConnection(host, port [, tls [, timeout]]) {#getConnection}

Establishes a new TCP connection with the given host on the given port.

- If the connection was previously saved using `saveConnection`, calling this will return that connection and un-save it.

- If the saved connection was closed, a new connection will be established instead.

***This method MUST be run in a coroutine***

#### Parameters {#getConnection-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| host  | string | The host which the established connection refers to. | ❌ |
| port  | number | The port that this connection should use when connecting to the host. | ❌ |
| tls   | boolean/[TLS Options](#tls-options) | The use of SSL/TLS encrypted protocol. <br> - Boolean value whether to use SSL/TLS cert or not.<br>- Table value to use SSL/TLS, with optional configurations. See [TLS Options](#tls-options) for the acceptable fields.| ✔ <br> Default: `false`. |
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

### request(method, url[, headers[, body[, options]]]) {#request}

Synchronously performs an HTTP(s) request after establishing a connection with the said host.

- If said connection was already established and is still alive, it will be used instead of establishing a new one.

***This method MUST be run in a coroutine***

#### Parameters {#request-parameters}

- **options** *(table  / number )* ***optional***:

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| method| string | An all uppercase HTTP method name. | ❌ |
| url   | url    | An HTTP(s) URL that the request should be sent to. | ❌ |
| headers| [http-header](#http-header) | The HTTP headers of the request.  | ✔ |
| body  | string | The request's payload (if needed).  | ✔ |
| options | [Request-Options](#request-options) / [Timeout](#timeout) | - If a number is supplied, this will act as the timeout to wait for the response before cancelling it out.<br>- If a table is supplied, this will act as a table to provide additional configurations. See [Request-Options](#request-options) for more details.  | ✔ |

#### Returns {#request-parameters}

| Name | Type   | Description |
|:-----|:------:|:------------|
| res  | [Response](#request-response) | A [Response](#request-response) structure representing the received response. |
| body | string/nil | The response payload (body) the server sent as string if any, otherwise nil. |

----

## Structures

Here are the data structures (tables usually) used by the library's functions, either as returns or as parameters. They were moved and linked to here to safe up space. Since they usually are repetitive and/or fairly big.

The `@string` syntax in this section links where the said structure was used TODO[9]. Any function referring to one of the listed structures will also link the said structure at its types definition for easier browsing.

### HTTP Header

A table structure representing an HTTP(s) header. The structure is a two-length array of strings, the first entry is the header-name, and the second entry is the header-value. See the [rfc2616 paper](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html) for more details about the officially acceptable HTTP headers.

#### The Structuring {#http-header-structuring}

**Full**: `{header-name, header-value}`.

  **Where**:

| Entry       | Type   | Description             |
|:------------|:------:|:------------------------|
| header-name | string | The name of the header. |
| header-value| string | The value of the header.|

**Examples**:

   - `{"Expires", "-1"}`.
   - `{"Accept", "text/plain"}`.

----

### Request / Response {#request-response}

Represents an HTTP(s) request/response including the headers, and general information about it. The exact data is highly dependent on the server/client side and therefore the docs cannot tell what values to expect or to not. If you want to know such information you should debug your code, or read the API manual of the said server.

#### The Structuring {#request-response-structuring}

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

   **Where**:

| Entry        | Type   | Description                |
|:-------------|:------:|:---------------------------|
| http_header  | [http-header](#http-header) | A sequence of [http-header](#http-header) structures each individually. |
| code         | number | The HTTP status code.      |
| reason       | string | The reason for getting the past status code (Reason-Phrase).        |
| version      | number | The version of the used HTTP(s) protocol as decimal number.|
| keepAlive    | boolean| Whether or not the connection should be kept alive. |

**Examples**:

{% raw %}

   - `{{"Content-Type", "text/html"}, {"Content-Length", "1587"}, code = 200, reason = "OK", version = 1.1, keepAlive = true}`.

{% endraw %}

----

### TCP Connection {#tcp-connection}

A table structure that represents a TCP connection (wrapped using coro-channel). The structure offers methods to directly read and write from the socket, and general details about the connection. Generally speaking you should only use those *directly* when it is the only way to accomplish what you need.

#### Available Fields {#tcp-connection-fields}

| Field | Type   | Description |
|:------|:------:|:------------|
| socket| [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle) | The TCP socket (handle) used to bind the established connection. |
| host | string | Same as the passed parameter `host` when establishing. |
| port | number | Same as the passed parameter `port` when establishing. |
| tls  | table / boolean | Same as the passed parameter `tls` when establishing. |
| read | function | TODO[3] |
| write| function | TODO[4] |
| updateEncoder | function | TODO[5] |
| updateDecoder | function | TODO[6] |
| reset | function | TODO[7] |

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
| ca | string / table | The x509 root certificates to check against. Defaults to a root certification (`root_ca.dat` file) when available. |
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
| hostname | string | The host name of the parsed URL (excluding port). |
| port     | number | The host port of the parsed URL (defaults to `80` for HTTP and `443` for HTTPS). |
| path     | string | Everything following the host of the parsed URL (including first `/`) |

----

### Timeout {#timeout}

A number value in milliseconds indicts how much time to wait for the response/request before canceling it out. If nothing is supplied [libuv](https://github.com/libuv/libuv) will timeout after an undefined amount of seconds.

**Examples**:

   - `1000`, waits for a one second.
   - `500`, waits for half of a second.

### Request Options {#request-options}

A table that could configure some of [Request](#request) behavior

#### The Structuring {#request-options-structuring}

**Full**:

```lua
   {
      timeout = (number),
      followRedirects = (boolean)
   }
```

   **Where**:

| Entry          | Type   | Description |
|:---------------|:------:|:------------|
| timeout        | [Timeout](#timeout) | See [Timeout](#timeout) for description. |
| followRedirects| boolean | whether or not to follow redirect requests. ***default***: `true`. |

----

# TODOs {#todos}

1. Complete the return table of [getConnection](#getConnection) function [4]/[5]/[6]/[7].
2. General examples and guides.

----
