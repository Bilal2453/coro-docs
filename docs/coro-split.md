# Documentations

A single-function library that takes multiple functions as input, and runs them in concurrent coroutines. [coro-split](https://github.com/luvit/lit/blob/master/deps/coro-split.lua) was originally made for [Lit](https://github.com/luvit/lit/), and published to be used by anyone.

We will call the only function return of this module `split`. You should always call this inside a coroutine.

----

## split(...) {#split}

Takes multiple functions (tasks) and concurrently run them in coroutines while yielding the current running coroutine called this, till all of the supplied tasks finishes.

The tasks will be called directly without wrapping them in protected mode, therefor any error raised in one of the provided tasks will be propagated, unless you wrap this in [pcall](https://www.lua.org/manual/5.4/manual.html#pdf-pcall) or [similar](https://www.lua.org/manual/5.4/manual.html#pdf-xpcall) functions.

### Parameters {#split-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| ...   | function | The tasks you want to concurrently run. |

### Returns {#split-returns}

| Return | Type   | Description |
|:------:|:------:|:------------|
| ...    | Any    | The returns of each task by the order you supplied them in, only one return per task is respected. |
