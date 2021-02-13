# Documentations

Unofficial docs for the module [coro-fs](https://github.com/luvit/lit/blob/master/deps/coro-fs.lua) version 2.2.4.

[coro-fs](https://github.com/luvit/lit/blob/master/deps/coro-fs.lua) is a library for filesystem manipulations while keeping the sync style of your code, making use of Lua coroutines.

This library is a great replacement for the Luvit built-in callback styled fs library, since it uses coroutines to keep the sync style of your code, without blocking the main event loop of luv, and most importantly, without the ugly callbacks! It is also somewhat simpler to use than the built-in fs library.

----

## Errors

All of the functions documented here will return a fail tuple in case of failure, that is, `nil, errMsg`, hence why all of the first returns have the `/nil`. The `errMsg` return is a string that looks something similar to `EEXIST: file already exists: foo`. Be aware that this only applies to operation failure, meaning, if you for example supply the wrong amount/type of parameters it WILL raise an error.

The first part of the error is the error code, those are defined by libuv and listed up by [man7](https://man7.org/linux/man-pages/man2/) (each fs method individually). The second part explains what went wrong exactly, and the last part explains what file/directory the error refers to.

----

## Functions

----

### mkdir(path[, mode]) {#mkdir}

Creates a new directory using the provided `path`.

The owner of the created directory __should__ be the effective user/group ID of said process, or it __could__ inherit the parent directory group-ID. The exact behavior is filesystem dependant.

***This method MUST be run in a coroutine***

#### Parameters {#mkdir-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path of the directory you want to create | ❌ |
| mode  | number | The inode bit mode at which to create said directory. Not implemented on Windows. | ✔ <br> Default: `511`. |

#### Returns {#mkdir-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### open(path[, flags[, mode]]) {#open}

Opens a file, or possibly creates one using the said file path.

The owner of a created file __should__ be the effective user/group ID of said process, or it __could__ inherit the parent directory group-ID. The exact behavior is filesystem dependant.

***This method MUST be run in a coroutine***

#### Parameters {#open-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path of the file to open/create. | ❌ |
| flags | string | The access mode of the opened file. See ***TODO*** for valid values. | ✔ <br> Default: `"r"`. |
| mode  | number | The file's inode mode to be applied when a file is created. Note that this only applies to future access of the newly created file. | ✔ <br> Default: `438`. |

#### Returns {#open-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| descriptor | number/nil | The file descriptor number of the opened file on success, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### unlink(path) {#unlink}

Deletes (unlinks) a file from the filesystem.

***This method MUST be run in a coroutine***


#### Parameters {#unlink-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| path  | string | The path of the file to be deleted/unlinked. |

#### Returns {#unlink-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### stat(path) {#stat}

Retrieves information about the file/directory pointed out by `path`.

***This method MUST be run in a coroutine***

#### Parameters {#stat-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| path  | string | The path to the file or directory you want to retrieve information about. |

#### Returns {#stat-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| info | table/nil | The information about said file/directory on success, see ***TODO***, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### lstat(path) {#lstat}

Identical to [stat](#stat), except if the path was for a symbolic link, the information will be about the link itself instead of its target.

***This method MUST be run in a coroutine***

#### Parameters {#lstat-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| path  | string | The path to the file or directory you want to retrieve information about. |

#### Returns {#lstat-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| info | table/nil | The information about said file/directory on success, see ***TODO***, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### fstat(fd) {#fstat}

Identical to [stat](#stat), except that instead of accepting a path to file/directory, this accepts a file descriptor.

***This method MUST be run in a coroutine***

#### Parameters {#fstat-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| fd    | number | The descriptor of the file you want to retrieve the information about. |

#### Returns {#fstat-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| info | table/nil | The information about said file/directory on success, see ***TODO***, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### symlink(target, path) {#symlink}

Creates a symbolic link (also known as soft link) at `path` that points up to `target`.

***WARNING***: There is currently a bug with this, it was fixed in the latest GH version but not yet published to Lit.
***TODO***: In the next release of coro-fs a new parameter will be added, document it when it is published to Lit.

***This method MUST be run in a coroutine***

#### Parameters {#symlink-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| target| string | The path of the file you want to link against. |
| path  | string | The path to where to create the said link at. |

#### Returns {#symlink-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### readlink(path) {#readlink}

Retrieves the target path (the content) of the said symbolic link `path`.

***This method MUST be run in a coroutine***

#### Parameters {#readlink-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| path  | string | The path to the symbolic link you want to retrieve its content. |

#### Returns {#readlink-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| cont | string/nil | The target path (the content) of the symlink on success, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### chmod(path, mode) {#chmod}

Changes the inode mode bits (e.g. the permissions) of said file.

***This method MUST be run in a coroutine***

#### Parameters {#chmod-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| path  | string | Path to the file you want to manipulate. |
| mode  | number | Decimal number representing the bit mode. |

#### Returns {#chmod-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### fchmod(fd, mode) {#fchmod}

Identical to [chmod](#chmod), except that instead of accepting a file path, you use a file descriptor.

***This method MUST be run in a coroutine***

#### Parameters {#fchmod-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| fd    | number | The descriptor of the file to manipulate. |
| mode  | number | Decimal number representing the bit mode. |

#### Returns {#fchmod-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### read(fd[, length[, offset]]) {#read}

Reads `length` bytes of `fd` file's contents.

***This method MUST be run in a coroutine***

#### Parameters {#read-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| fd    | number | The descriptor of the file to read from. | ❌ |
| length  | number | How many bytes should be read from the file. | ✔ <br> Default: `49152`. |
| offset | number | How many bytes should be sought before start reading. `-1` means "Use current file position". |  ✔ <br> Default: `-1`. |

#### Returns {#read-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| data | string/nil | The read contents on success, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### write(fd, data[, offset]) {#write}

Writes data to file using its descriptor `fd`.

***This method MUST be run in a coroutine***

#### Parameters {#write-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| fd    | number | The descriptor of the file to write data to. | ❌ |
| data  | string | The data you want to write to the file as string. | ❌ |
| offset | number | How many bytes should be sought before writing the data. `-1` means "Use the current file position". |  ✔ <br> Default: `-1`. |

#### Returns {#write-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### close(fd) {#close}

Closes the opened file `fd`, so `fd` don't refer to anything anymore.

***This method MUST be run in a coroutine***

#### Parameters {#close-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| fd    | number | The descriptor of the file to be closed.

#### Returns {#close-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### access(path[, mode]) {#access}

Checks whether the current process have permissions to access the file/directory `path`.

***This method MUST be run in a coroutine***

#### Parameters {#access-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path to the file/directory you want to check against. | ❌ |
| mode  | number/string | The accessibility check, number to specify the bit permissions mode, string to specify access mode (`"R"` for read, `"W"` for write, `"X"` for execute). | ✔ <br> Default: `""`. |

#### Returns {#access-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| perm | boolean/nil | Boolean indicts whether you have permissions to access said file/directory or not on success, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### rename(path, newPath) {#rename}

Renames a file or a directory to `newPath`. Can be also used to move a file/directory just by specifying `newPath` to the target path to move files into.

***This method MUST be run in a coroutine***

#### Parameters {#rename-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| path    | string | The path to the file or directory you want to rename/move. |
| newPath | string | The new path/name of the renamed/moved file or directory. |

#### Returns {#rename-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### rmdir(path) {#rmdir}

Removes and deletes an empty (and only an empty) directory. For recursive remove see [rmrf](#rmrf).

***This method MUST be run in a coroutine***

#### Parameters {#rmdir-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| path  | string | The path to the directory you want to delete. |

#### Returns {#rmdir-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### rmrf(path) {#rmrf}

Tries to recursively delete `path`, while handling most possible scenarios. Most suitable for deleting non-empty directories, or when you just want something to be deleted without having to handle additional exceptions.

- If the path is a directory its contents will be deleted first then it will be removed.
- If the path is a file it will be deleted.
- If the path is a symbolic link it will be unlinked.

***WARNING***: You cannot undo this!

***This method MUST be run in a coroutine***

#### Parameters {#rmrf-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| path  | string | The path to the object you want to remove. |

#### Returns {#rmrf-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### scandir(path) {#scandir}

An iterator that iterates a directory for files and sub-directories.

***This method MUST be run in a coroutine***

#### Parameters {#scandir-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| path  | string | The directory path to be scanned. |

#### Returns {#scandir-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| entry| table/nil | A table that contains two fields `name` and `type` on success, otherwise nil. | Always |
| err  | string | A string explaining what went wrong when executing said operation. | Failure Only |

***TODO***: Example.

----

### readFile(path) {#readFile}

Fully reads a file and returns its contents as a single string.

***NOTE***: In some rare and system specific cases, only a chunk of the file will be read.

***This method MUST be run in a coroutine***

#### Parameters {#readFile-parameters}

| Param | Type   | Description |
|:------|:------:|:------------|
| path  | string | The path to the file you want to read. |

#### Returns {#readFile-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| data | string/nil | The contents of the file on success, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### writeFile(path, data[, mkdir]) {#writeFile}

Writes data to file `path` in a one go.

***This method MUST be run in a coroutine***

#### Parameters {#writeFile-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path to the file you want to write into. | ❌ |
| data  | string | The data you want to write into the file as string. | ❌ |
| mkdir | boolean| Whether you want coro-fs to create any missing directories in the provided `path` or not. | ✔️ <br> Default: `false`. |

#### Returns {#writeFile-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### mkdirp(path[, mode]) {#mkdirp}

Recursively creates missing directories in a path with the said bit mode.

For example `mkdir("./a/b/c/")` this call will create a directory `a` that contains directory `b` if they don't already exists, etc.

***This method MUST be run in a coroutine***

#### Parameters {#mkdirp-parameters}

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path that might contains the missing directories to be created. | ❌ |
| mode  | number | Decimal number of the indoe mode bits (e.g. its permissions). | ✔️ |

#### Returns {#mkdirp-returns}

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### chroot(base) {#chroot}

***TODO***: Document this.

*This method does not require running in a coroutine*
