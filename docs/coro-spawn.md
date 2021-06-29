---
layout: doc
---

# Documentation

Unofficial docs for the [coro-spawn](https://github.com/luvit/lit/blob/master/deps/coro-spawn.lua) module, version 3.0.3.

[coro-spawn](https://github.com/luvit/lit/blob/master/deps/coro-spawn.lua) is single-function module that provides manipulating child-process in a sync style of code using Lua coroutines.

We will call the only function return of this module `spawn`.

----

## Functions

----

### spawn(path, options) {#spawn}

Spawns a process of an executable at `path` using the said `options`.

Note: The `options` parameter is NOT optional, if you don't want to provide any options just pass an empty table.

*This method does not require running in a coroutine*

#### Parameters {#spawn-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| path  | string | The path of the executable (defined by system) to be spawned. |
| options | table| Various options to control the process flow. See [Spawn Options](#spawn-options). |

#### Returns {#spawn-returns}

| Return | Type   | Description | Provided On |
|:-------|:------:|:------------|:-----------:|
| result | [Process](#process)/nil | Immediately returns with the proper streams and wrappers representing the process on success, otherwise nil. You probably want to call `waitExit` after this, see [Process Fields](#process-fields). | Always |
| err    | string | A string explaining what went wrong when trying to spawn the process. | Failure Only |

#### Examples {#spawn-examples}

- List the previous directory contents to stdout

```lua
local proc, err = spawn('dir', { -- Spawn a process of the executable command "dir"
  stdio = {0, 1, 2}, -- Use same stdio as the parent process
  cwd = "../" -- Set the working directory to the previous directory
})
if not proc then -- Any errors?
  print("Something went wrong!\n" .. err); return
end
local code, sig = proc.waitExit() -- Wait until the executed command finishes
print("\nFinished! exit-code: " .. code .. " exit-signal: " .. sig)
```

----

## Structures

### Spawn Options {#spawn-options}

A table that contains various configurations to control the process flow. All of the fields are optional.

#### Fields {#spawn-options-fields}

| Field | Type   | Description |
|:-----:|:------:|:------------|
| stdio | table  | An array of listed values from index 1 to 3 where each index represents stdin, stdout, stderr in order.<br>- If the value is a boolean, true means automatically create and assign a [pipe](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle) for the corresponding handle, false and nil to ignore.<br>- If the value is a [pipe](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle) it will be assigned to the corresponding handle index as read-write stream.<br>- If it's a number, then the child process inherits that same zero-indexed fd from the parent process. |
| args | table  | The command line arguments as a list of strings. |
| env  | table  | Set environment variables for the new process. |
| cwd  | string | Set the current working directory for the sub-process. |
| uid  | number | Set the child process' user id. |
| gid  | number | Set the child process' group id. |
| verbatim | boolean | If true, don't perform any kind of escaping when converting the argument list into a command line string. This option is only implemented on Windows systems. |
| detached | boolean | If true, spawn the child process in a detached state, this will make it a process group leader, and will effectively enable the child to keep running after the parent exits. |
| hide | boolean | If true, hide the subprocess console window that would normally be created. This option is only implemented on Windows systems. |

----

### Stdio {#stdio}

A table structure that represents a stdio handle, either a stdin, stdout or stderr, depending on the context.

#### Fields {#stdio-fields}

| Field | Type   | Description | Provided On |
|:-----:|:------:|:------------|:-----------:|
| handle| [pipe](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle) | The uv pipe handle used to bind said stream. | Always |
| read  | function | Reads data from a readable stream. See [coro-channel reader](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader). | stdout & stderr Only |
| write | function | Writes data to a writable stream. See [coro-channel writer](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#writer). | stdin Only |

----

### Process {#process}

A table structure that represents the the spawned process.

#### Fields {#process-fields}

| Field | Type   | Description |
|:-----:|:------:|:------------|
| handle| [uv_process_t](https://github.com/luvit/luv/blob/master/docs.md#uv_process_t--process-handle) | The handle object of the spawned process. |
| pid   | number | The PID process number assigned to the spawned process. |
| waitExit | function | Calling this will yield the current coroutine and resume it when the process finishes, will immediately returns if the process has already finished. Returns `exitCode` and `exitSignal`. |
| stdin | [stdio](#stdio)/nil | The used stdin stream, or nil if no handle is created. |
| stdout| [stdio](#stdio)/nil | The used stdout stream, or nil if no handle is created. |
| stderr| [stdio](#stdio)/nil | The used stderr stream, or nil if no handle is created. |

----
