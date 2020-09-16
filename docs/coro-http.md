# About
These are unofficial docs for the Luvit library [coro-http](https://github.com/luvit/lit/blob/master/deps/coro-http.lua) that was originally made by Tim Caswell, and the help of the contributors.

It can be used as a great replacement for the built-in http library in Luvit for those who don't like the callback style, this library uses Lua's coroutines to keep the sync style with optional coroutines wrapping for asynchronous execution. And where it also handles things the standard http library does not handle, such as chunking.

These docs will guide you through all the available methods and their usage. If you find any wrong documentation, confusing wording, or even typos, please open an issue or even a PR!

Many thanks for [@trumedian](https://github.com/truemedian) for helping out from behind the scenes by correcting many invalid infos, better wording, and pointing out many typos.

# TODOs
1. Complete the TLS parameter table entries for #getConnection [2].
2. Complete the return table of the #getConnection function [4]/[5]/[6]/[7].
3. Document #request `headers` parameter defaults [8].
5. General examples and guides.
6. Execute this idea [9].

# Documentations

## Functions

### createServer (host, port, onConnect)
Creates a new server instance and asynchronously binds it to host:port.

#### Parameters
- **host** *(string)*: The host which the server corresponds to.
- **port** *(number)*: The opened port for the server to listen to.
- **onConnect** *(function)*: A callback that will be asynchronously called every time a new connection is established to the server.
The callback has the following parameters:

| Param | Type   | Description |
|:------|:------:|:------------|
| req   | table ([Request](#request--response)) | The request headers and status. |
| body  | string | The provided request's payload as string, empty string in case nothing is provided. |
| socket| [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle) / [uv_pipe_t](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle)| The socket that the connection was bound to. |

---

### parseUrl (url)
Parses the given string representing an HTTP(s) URL to a Lua table.

#### Parameters:
- **url** *(string)*: The URL that should be parsed.
  - Must be a valid HTTP(s) URL or an error will be raised.

#### Returns:
1. *(table [Parsed URL](#parsed-url))*: The parsed URL as a Lua table.
   - See [Parsed URL](#parsed-url) structure for more details.

---

### getConnection (host, port [, tls [, timeout]])
Establishes a new TCP connection with the given host on the given port.

  – If the connection was saved previously using `saveConnection`, calling this will return that saved connection and un-save it.

  – If the saved connection was closed, a new connection will be established instead.

#### Parameters:
- **host** *(string)*: The host which the established connection refers to.
- **port** *(number)*: The port that this connection should use to connect to the host.
- **tls** *(boolean / table [TLS Options](#tls-options))* ***optional***: The use of SSL/TLS encrypted protocol. *default*: `false`.
  - Boolean value whether to use SSL/TLS cert or not.
  - Table value to use SSL/TLS, with optional configurations. See [TLS Options](#tls-options) for the acceptable fields.
- **timeout** *(number [Timeout](#Timeout))* ***optional***: How much time to wait for the response before canceling the request out. *default*: `nil`.

#### Returns
1. *(table [TCP Connection](#tcp-connection))*: The established connection.
   - See [TCP Connection](#tcp-connection) structure for more information.

---

### saveConnection (connection)
Saves a pre-established [TCP connection](#tcp-connection) to be used later instead of establishing a new one.

  – If the passed connection is already closed nothing will be saved.

#### Parameters:
- **connection** *(table [TCP Connection](#tcp-connection))*: A table representing a TCP connection returned by `getConnection`.

---

### request  (method, url[, headers[, body[, timeout]]])
Synchronously performs an HTTP(s) request after establishing a connection with the said host.

#### Parameters
-  **method** *(string)*: An all uppercase HTTP method.

- **url** *(string)*: An HTTP(s) URL that the request should be sent to.

- **headers** *(table [http-header](#http-header))* ***optional***: The request headers. *default*: TODO[8].

- **body** *(string)* ***optional***: The request's payload (if needed). *default*: `nil`.

- **timeout** *(number [Timeout](#Timeout))* ***optional***: How much time to wait for the response before canceling the request out. *default*: `nil`.

#### Returns

1. *(table [Response](#request--response))*: The response headers and status. See [Response](#request--response) structure for more details.

2. *(string)*: The response payload (body) as string.
   - This is whatever the server responds with.

---

## Structures
Here are the data structures (tables usually) used by the library's functions, either as returns or as parameters. They were moved and linked to here to safe up space. Since they usually are repetitive and/or fairly big.

The `@string` syntax in this section links where the said structure was used TODO[9]. Any function referring to one of the listed structures here will also give a link at its type definition for easier browsing.

### HTTP Header
A table structure representing an HTTP(s) header. The structure is a two-length strings array, the first entry is the header-name, and the second entry is the header-value, both as string. See the [officially available HTTP(s) 1.1 fields](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html) for more information about headers.

#### The Structuring
**Full**: `{header-name, header-value}`. 

  **Where**:
| Entry        | Type   | Description       |
|:------------ |:------:|:----------------- |
| header-name  | string | The name of the header.  |
| header-value | string | The value of the header. |

**Examples**:
   - `{"Expires", "-1"}`.
   - `{"Accept", "text/plain"}`.

---

### Request / Response
Represents an HTTP(s) request/response including the headers, and general information about it. The exact data is highly dependent on the server/client side and therefore the docs cannot tell what values to expect or to not. If you want to know such information you should debug your code, or read the API manual of the said server.

#### The Structuring
**Full**:
```lua
  {
    http-header...,
    code = (number),
    reason = (string),
    version = (number),
    keepAlive = (boolean),
  }
```

   **Where**:
| Entry        | Type   | Description       |
|:------------ |:------:|:----------------- |
| http-header  | table ([http-header](#http-header)) | A sequence of [http-header](#http-header) structures. |
| code         | number | The HTTP status code.  |
| reason       | string | The reason behind getting the past status code (Reason-Phrase).
| version      | number | The version of the used HTTP(s) protocol as decimal number.|
| keepAlive    | boolean| Whether or not the connection should be kept alive. |

**Examples**:
   - `{{"Content-Type", "text/html"}, {"Content-Length", "1587"}, code = 200, reason = "OK", version = 1.1, keepAlive = true}`.

---

### TCP Connection
A table that represents a wrapped TCP connection (wrapped using coro-channel), it contains many useful things when you want to directly work with the connection and the socket. Usually you only should touch this directly when you do know what you are dealing with.

#### Available Fields

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

### TLS Options
Here are the available options and fields for configuring SSL/TLS connections.

#### Acceptable Fields
| Field | Type   | Description |
|:------|:------:|:------------|
| protocol | string | The transport-layer-secure protocol to use. Supported values depends on lua-openssl version, openssl TLS 1.3 compatible supports: `TLS` (default) or `DTLS`. LibreSSL and others uses `SSLv23` by default and supports other versions.
| ciphers | string | The encryption algorithm to encrypt data with, value **MUST** be a valid cipher suite string. Defaults are `TLS_AES_128_GCM_SHA256:TLS_AES_128_CCM_SHA256` for TLS 1.3, `ECDHE-RSA-AES128-SHA256:AES128-GCM-SHA256` for LTS 1.2, `RC4:HIGH:!MD5:!aNULL:!EDH` for LTS 1.0. |
| key | string | The PEM key of the supplied certification (if `cert` field is passed). |
| cert | string | The SSL/TLS x509 certification used for the handshake as string. Used alongside with field `key` or it gets ignored. |
| ca | string / table | The x509 certificates/CRLs to store. Defaults to a root certification (`root_ca.dat` file) when available. |
| insecure | boolean | TODO[2] |

*All of the fields are optional and should only be touched when you know what you are dealing with.*

### Parsed URL
A parsed URL is usually returned by [parseUrl](#parseurl-url) as a table that represents an HTTP(s) URL.

#### Available Fields
| Fields   | Type   | Description |
|:-------- |:------:|:------------|
| tls      | boolean| Whether or not the parsed URL uses TLS/SSL (HTTPS). |
| host     | string | The authority of the parsed URL (hostname:port). |
| hostname | string | The host name of the parsed URL (excluding port). |
| port     | number | The host port of the parsed URL (defaults to `80` for HTTP and `443` for HTTPS). |
| path     | string | Everything following the host of the parsed URL (including first `/`) |


### Timeout
A number value in milliseconds indicts how much time to wait for the response/request before canceling it out. If nothing is supplied [libUV](https://github.com/libuv/libuv) will timeout after an undefined amount of seconds.

**Examples**:
   - `1000`, waits for a one second.
   - `500`, waits for half of a second.
---
