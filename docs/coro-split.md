---
layout: doc
---

# Documentation

Unofficial docs for the [coro-split](https://github.com/luvit/lit/blob/master/deps/coro-split.lua) module, version 2.0.0.

[coro-split](https://github.com/luvit/lit/blob/master/deps/coro-split.lua) is single-function module that takes multiple functions as input, and runs them in concurrent coroutines.

We will call the only function return of this module `split`. You should always call this inside a coroutine.

----

## Functions

----

### split(...) {#split}

Takes multiple functions (tasks) and concurrently run them in coroutines while yielding the coroutine that called this, then resume it when all of the supplied tasks finishes.

The tasks will be called directly without running them in protected mode, therefor any errors raised out by any of the provided tasks will be propagated, unless you call `split` using [pcall](https://www.lua.org/manual/5.4/manual.html#pdf-pcall) or [similar](https://www.lua.org/manual/5.4/manual.html#pdf-xpcall) functions.

***This method MUST be run in a coroutine***

#### Parameters {#split-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| ...   | function | The tasks you want to concurrently run. No limit defined by coro-split. |

#### Returns {#split-returns}

| Return | Type   | Description |
|:------:|:------:|:------------|
| ...    | Any    | The returns of each task by the order you supplied them in, only one return per task is respected. |

----
