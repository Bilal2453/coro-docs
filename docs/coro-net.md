# Documentations

Unofficial docs for the module [coro-net](https://github.com/luvit/lit/blob/master/deps/coro-net.lua) version 3.2.1.

[coro-net](https://github.com/luvit/lit/blob/master/deps/coro-net.lua) is a library for TCP and pipes manipulations, with optional secure-layer support while keeping the sync code style.

----

## Functions

----

### connect(options) {#connect}

Establishes a connection with a TCP handle, or with a pipe handle.

- Both IPv4 and IPv6 can be used for TCP connections.
- Pipe handles created by this cannot be used for IPC.

#### Parameters {#connect-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| options | boolean/table | See [Options](#options) for details. |

#### Returns {#connect-returns}

**On Success:**

| Name | Type   | Description |
|:-----|:------:|:------------|
| read | function | The stream read wrapper, if one of `options.decode` or `options.scan` is supplied; this would be a [coro-wrapper's adapter](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html#functions), otherwise it'd be [coro-channel's reader](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader). Adapters can be stacked up if multiple options are passed. |
| write| function | The stream write wrapper, if `options.encode` is supplied; this would be a [coro-wrapper's adapter](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html#encoder), otherwise it'd be [coro-channel's writer](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#writer). |
| socket | [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle)/[uv_pipe_t](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle) | The [stream](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) handle of the said established connection. |
| updateDecoder | function | A function that takes a single function argument and sets `options.decode` to that input. |
| updateEncoder | function | A function that takes a single function argument and sets `options.encode` to that input. |
| close | function | Waits for any queued operations to finish and closes the socket completely using [coro-channel's closer](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#closer). |

**On Failure:**

| Name | Type  | Description |
|:-----|:-----:|:------------|
| fail | nil   | If the first return is nil, that means failure has occurred. |
| err  | string| The error message explaining what went wrong. |

----

### createServer(options, onConnect) {#createServer}

Creates and binds a server instance to a TCP or pipe handle, while asynchronously executing the supplied callback.

#### Parameters {#createServer-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| options | table | See [Options](#options) for details. |
| onConnect | function | See [Callback](#createServer-callback-onConnect) for details. |

#### Returns {#createServer-returns}

| Name | Type   | Description |
|:-----|:------:|:------------|
| server | [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle)/[uv_pipe_t](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle) | The handle representing the bound server. |

#### Callback: onConnect(read, write, socket, updateDecoder, updateEncoder) {#createServer-callback-onConnect}

A callback that'd be called every time a new incoming connection is received.

**Callback Parameters:**

| Param | Type     | Description |
|:------|:--------:|:------------|
| read  | function | The stream read wrapper. |
| write | function | The stream write wrapper. |
| socket | [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle)/[uv_pipe_t](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle) | The [stream](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) handle the server was bound to. |
| updateDecoder | function | A function that takes a single function argument and sets `options.decode` to that input. |
| updateEncoder | function | A function that takes a single function argument and sets `options.encode` to that input. |

**Returns:**

| Return | Type     | Description |
|:------:|:--------:|:------------|
| success| boolean  | Whether or not your callback has succeeded. |
| failure| string   | An error message that will be print to stdout explaining what went wrong, without raising an actual error. |

----

### makeCallback([timeout]) {#makeCallback}

**TODO**

----

## Structures

### Options {#options}

A string-indexed table structure offering different configurations of how coro-net should be interacting and behaving.

#### Fields {#options-field}

| Field | Type   | Description | Required |
|:-----:|:------:|:------------|:--------:|
| path  | string | The pipe name the stream refers to, only relevant if `host` and `port` aren't provided. | Either `path` or `port`/`host` must be provided. |
| host  | string | The hostname a TCP connection should be referring to. If `port` is provided but `host` isn't, the default value's used. | Either `path` or `port`/`host` must be provided.<br> Default: `"127.0.0.1"`. |
| port  | number | The port of the TCP host the connection refers to. | Required if `host` is provided. |
| tls   | boolean/table | Whether or not to use secure-layer on top of the connection. A table value for using secure-layer with further configurations, see [options](#options) for more details. | Optional |
| timeout | number | How many millisecond should be wait before canceling the connection out. An undefined amount of seconds is used if nothing is provided. | Optional |
| socktype | string | The socket type should be used for outgoing connections. | Optional<br>Default: `"stream"`. |
| family| string | The outgoing connection family (type). | Optional<br>Default: `"inet"`. |
| scan  | function | A [merger](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html#merger) adapter to be used around the [read](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader) wrapper. | Optional |
| decode| function | A [decoder](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html#decoder) adapter to be used around the [read](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader) wrapper. | Optional |
| encode| function | An [encoder](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html#encoder) adapter to be used around the [write](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#writer) wrapper. | Optional |

----

### TLS Options {#tls-options}

An optional string-indexed table that offers further configurations over the used secure-layer.

Note: when using [createServer](#createServer) you must supply `cert` and `key`.

#### Fields {#tls-options-fields}

| Field | Type   | Default | Description |
|:------|:------:|:-------:|:------------|
| protocol | string | `"TLS"` or `"SSLv23"`. | The transport-layer-secure protocol to use. Supported values depends on lua-openssl version, openssl TLS 1.3 compatible supports: `TLS` (default) or `DTLS`. LibreSSL and others uses `SSLv23` by default, and may support other protocols. |
| ciphers | string | `"TLS_AES_128_GCM_SHA256:TLS_AES_128_CCM_SHA256"` for TLS 1.3, `"ECDHE-RSA-AES128-SHA256:AES128-GCM-SHA256"` for LTS 1.2, `"RC4:HIGH:!MD5:!aNULL:!EDH"` for LTS 1.0. | The encryption algorithm to encrypt the stream with, value **MUST** be a valid cipher suite string. |
| key | string | | The PEM key of the supplied certification (if `cert` field is passed). | |
| cert | string | The SSL/TLS x509 certification used for the handshake as string. Used alongside with field `key` or it gets ignored. |
| ca | string/table | Defaults to a root certification CA (`root_ca.dat` file) when available. | The x509 root certificates authority to check against. |
| insecure | boolean | `false`. | Weather or not to accept invalid certificates on handshakes. |

----
