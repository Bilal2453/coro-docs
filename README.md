# coro-docs

Documentation for the [Luvit](https://luvit.io/) coro-* libraries.

The modules listed in here implement a way to do asynchronous code but with Lua coroutines, to achieve a cleaner code style. They do this by yielding the currently running coroutine and resuming it once a call is done (instead of calling a user-defined callback), which allows the event-loop to still do other things in the background.

Take for example this example:

coro-http:

```lua
local http = require("coro-http")
local res, body = http.request("GET", "http://www.google.com/") -- yields until request is done
print(res.code, body)
```

vs

built-in http:

```lua
local http = require("http")
http.get("http://www.google.com/", function(res)
  print(res.code)
  res:on("data", print)
end)
print("this is printed before the request is made")
```

With the coro-http example, it's much easier and much more ergonomic to use, and you may still make multiple requests concurrently by running multiple coroutines.


## Index

- [coro-channel](https://bilal2453.github.io/coro-docs/docs/coro-channel.html)
- [coro-fs](https://bilal2453.github.io/coro-docs/docs/coro-fs.html)
- [coro-http](https://bilal2453.github.io/coro-docs/docs/coro-http.html)
- [coro-net](https://bilal2453.github.io/coro-docs/docs/coro-net.html)
- [coro-spawn](https://bilal2453.github.io/coro-docs/docs/coro-spawn.html)
- [coro-split](https://bilal2453.github.io/coro-docs/docs/coro-split.html)
- [coro-websocket](https://bilal2453.github.io/coro-docs/docs/coro-websocket.html)
- [coro-wrapper](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html)

## A coroutine is required

Most calls in the listed modules require a running coroutine (ones that don't are documented), because of that Luvit (starting with version 2.18) wraps the main chunk in a coroutine for you making it much simpler. You might still run into cases where you need to manually wrap a call in a coroutine, notably when you run something in a luv callback because that's executed outside of the main scope.
In other words, when you get an error such as `attempt to yield across C-call boundary` or `Cannot make HTTP request outside of a coroutine`, you want to add the following to your code:

```lua
coroutine.wrap(function()
  -- the code to handle the call, such as http.request()
end)()
-- or alternatively directly coroutine.wrap(http.request)()
```

## Thread-pool & Event-loop

Keep in mind that when dealing with synchronous vs asynchronous APIs there is a tradeoff to make, you don't always want the asynchronous calls and vise versa.

First of all, Luvit (and by extension, the coro libraries) utilize the libuv backend to do the synchronous/asynchronous calls. For performance reasons libuv spawns a number of threads and stores them in a thread-pool for when they are needed.

> **Tip**:
> by default 4 threads are spawned, and may be increased by setting `UV_THREADPOOL_SIZE` environment variable.

Now when you need to do a synchronous blocking call (such as a `fs.readFileSync`), instead of blocking the main thread where the event-loop handle everything, it offloads this job to one of the available threads in the thread-pool, which lets you run multiple blocking operations without halting the entire program.

This is notably useful when the synchronous interface offers a much better performance compared to its asynchronous counterpart, a good example of this is with the built-in `fs` module, while something like `fs.statSync` will block a thread, it's actually faster to call than `fs.stat` (async version).
When you know for sure that an operation won't take much time (such as when calling `fs.stat`) you may instead opt for using the synchronous version of the call which takes almost no time to finish (unlike the async version of the call).

It is important to choose the right interface for your usage, especially when exposing a public web service, for example if a `fs.writeSync` call will take a long time to finish (like writing a large file), it's possible to DoS your server by spamming this call and over-whelming your thread-pool (which can't grow in size dynamically) preventing other synchronous calls from running, in this case opting for asynchronous is better as now you offload it to the Operating System queue instead.
