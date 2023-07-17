---
layout: doc
---

# coro-split

Documentation for the [coro-split](https://github.com/luvit/lit/blob/master/deps/coro-split.lua) module, version 2.0.2.

[coro-split](https://github.com/luvit/lit/blob/master/deps/coro-split.lua) is single-function module that takes multiple functions as input, wraps each of them in a coroutine and runs it, and yields until all of the provided tasks are completed.

Throughout the documentation, we will refer to the function return of this module as `split`. And it is obtained by calling `require("coro-split")`.

### Installation

```sh
lit install creationix/coro-split
```
[On Lit search.](https://luvit.io/lit.html#coro-split)

----

## Functions

----

### split(...) {#split}

Takes multiple functions (tasks) and runs them in coroutines while yielding the coroutine that made this call, then resumes it when all of the supplied tasks finishes.

Returns the first return of every task.

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

#### Examples {#split-examples}

- Concurrent Filesystem manipulations

```lua
local fs = require('fs')
split(
  function() -- Task 1: Prints the current directory contents
    p(1)
    local thread = coroutine.running()
    fs.readdir('./', function(...)
      coroutine.resume(thread, ...)
    end)
    p(2, coroutine.yield())
  end,
  function() -- Task 2: Print if we can access current directory
    p(3)
    local thread = coroutine.running()
    fs.access('./', function(...)
      coroutine.resume(thread, ...)
    end)
    p(4, coroutine.yield())
  end,
  function() -- Task 3: Prints the current directory stats
    p(5)
    local thread = coroutine.running()
    fs.stat('./', function(...)
      coroutine.resume(thread, ...)
    end)
    p(6, coroutine.yield())
  end
)
```

TODO:  This example is awful and meaningless, rewrite is needed.

----
