# Documentations

These are unofficial documentations for the Luvit coroutines based library [coro-fs](https://github.com/luvit/lit/blob/master/deps/coro-fs.lua) version 2.2.4, it was originally developed by [Tim Caswell](https://github.com/creationix) for [Lit](https://github.com/luvit/lit/), but can also be used by anyone.

This library is a great replacement for the Luvit built-in callback styled fs library, since it uses coroutines to keep the sync style of your code, without blocking the main event loop of luv, and mos importantly, without the ugly callback! It is also somewhat simpler to use than the built-in fs library.

The only downside to this library is that every call you make has to be inside of a coroutine, whether it is directly wrapped, or wrapped at a higher level. Since any call will always try to yield the current running coroutine (except `fs.chroot`).

----

## Errors

All of the functions documented here will return a fail tuple in case of failure, that is, `nil, errMsg`, hence why all of the first returns have the `/nil`. The `errMsg` return is a string that looks something similar to `EEXIST: file already exists: foo`. Be aware that this only applies to operation failure, meaning, if you for example supply the wrong amount/type of parameters it WILL raise an error.

The first part of the error is the error code, those are defined by libuv and listed up by [man7](https://man7.org/linux/man-pages/man2/) (each fs method individually). The second part explains what went wrong exactly, and the last part explains what file/directory the error refers to.

----

## Functions

----

### mkdir(path[, mode])

Creates a new directory using the provided `path`.

The owner of the created directory __should__ be the effective user/group ID of said process, or it __could__ inherit the parent directory group-ID. The exact behavior is filesystem dependant.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path of the directory to create | ❌ |
| mode  | number | The inode bit mode at which to create said directory. Not implemented on Windows. | ✔ Default: `511`. |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### open(path[, flags[, mode]])

Opens a file or possibly creates one using the said file path.

The owner of a created file __should__ be the effective user/group ID of said process, or it __could__ inherit the parent directory group-ID. The exact behavior is filesystem dependant.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path of the file to open/create. | ❌ |
| flags | string | The access mode of the opened file. See ***TODO*** for valid values. | ✔ Default: `"r"`. |
| mode  | number | The file's inode mode to be applied when a file is created. Note that this only applies to future access of the newly created file. | ✔ Default: `438`. |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| descriptor | number/nil | The file descriptor number of the opened file on success, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### unlink(path)

Deletes (unlinks) a file from the filesystem.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path of the file to be deleted/unlinked. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### stat(path)

Retrieves information about the file/directory pointed out by `path`.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path to the file or directory you want to retrieves information about. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| info | table/nil | The information about said file/directory on success, See [TODO], nil otherwise. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### lstat(path)

Identical to [stat](#stat), except if the path was for a symbolic link, the information will be about the link itself instead of its target.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path to the file or directory you want to retrieves information about. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| info | table/nil | The information about said file/directory on success, See [TODO], nil otherwise. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### fstat(fd)

Identical to [stat](#stat), except that instead of a path to file/directory, this accepts a file descriptor.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| fd    | number | The descriptor of the file you want to get the information about. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| info | table/nil | The information about said file/directory on success, See [TODO], nil otherwise. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### symlink(target, path)

Creates a symbolic link (also known as soft link) at `path` that points to `target`.

***WARNING***: There is currently a bug with this, that was fixed in the latest GH version.
***TODO***: In the next release of coro-fs a new parameter will be added, document it when it is published to Lit.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| target| string | The path of the file you want to link against. | ❌ |
| path  | string | The path to where to create the said link at. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### readlink(path)

Retrieves the target path (the content) of the said symbolic link `path`.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path to the symbolic link you want to retrieve its target. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| cont | string/nil | The target path (the content) of the symlink on success, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### chmod(path, mode)

Changes the mode bits (e.g. the permissions) of said file.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | Path to the file you want to manipulate. | ❌ |
| mode  | number | Decimal number representing the bit mode. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### fchmod(fd, mode)

Identical to [chmod](#chmod), except that instead of a path to a file, you use a file descriptor.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| fd    | number | The descriptor of the file to manipulate. | ❌ |
| mode  | number | Decimal number representing the bit mode. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### read(fd[, length[, offset]])

Reads `length` bytes of `fd` file's contents.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| fd    | number | The descriptor of the file to read from. | ❌ |
| length  | number | How many bytes should be read from the file. | ✔ Default: `49152`. |
| offset | number | How many bytes should be sought before start reading. `-1` means "Use current file position". |  ✔ Default: `-1`. |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| data | string/nil | The read contents on success, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### write(fd, data[, offset])

Writes data to file using its descriptor `fd`.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| fd    | number | The descriptor of the file to write data to. | ❌ |
| data  | string | The data you want to write to the file as string. | ❌ |
| offset | number | How many bytes should be sought before start writing. `-1` means "Use current file position". |  ✔ Default: `-1`. |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### close(fd)

Closes the opened file descriptor `fd`, so it doesn't refer to anything anymore.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| fd    | number | The descriptor of the file to be closed. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### access(path[, mode])

Checks whether the calling process have permissions to access the file/directory `path`.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path to the file/directory you want to check against. | ❌ |
| mode  | number/string | The accessibility check, number to specify the bit permissions mode, string to specify access mode (`R` for read, `W` for write, `X` for execute.). | ✔ Default: `""`. |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| perm | boolean/nil | Boolean indicts whether you have permissions to access said file/directory or not on success, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### rename(path, newPath)

Renames a file or a directory to `newPath`. Can be also used to move a file/directory just by specifying `newPath` to the target path to move files into.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path    | string | The path to the file or directory you want to rename(move). | ❌ |
| newPath | string | The new path/name of the renamed/moved file or directory. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### rmdir(path)

Removes and deletes an empty (and only an empty) directory. For recursive remove see [rmrf](#rmrf).

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path to the directory you want to delete. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### rmrf(path)

Tries to recursively delete `path`, while handling most possible scenarios. Most suitable for deleting non-empty directories, or when you just want something to be deleted without having to handle additional exceptions.

- If the path is a directory its contents will be deleted first then it will be removed.
- If the path is a file it will be deleted.
- If the path is a symbolic link it will be unlinked.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path to the object you want to remove. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### scandir(path)

An iterator that iterates a directory for files and sub-directories.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The directory path to be scanned. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| entry| table/nil | A table that contains two fields `name` and `type` on success, otherwise nil. | Always |
| err  | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### readFile(path)

Fully reads a file and returns its contents as single string.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path to the file you want to read. | ❌ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| data | string/nil | The contents of the file on success, otherwise nil. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### writeFile(path, data[, mkdir])

Writes data to file `path` in a one go.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path to the file you want to write into. | ❌ |
| data  | string | The data you want to write into the file as string. | ❌ |
| mkdir | boolean| Whether you want coro-fs to create any missing directories in the provided `path` or not. | ✔️ Default: `false`. |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### mkdirp(path[, mode])

Recursively creates a path with the said bit mode. For example `mkdir("./a/b/c/")` this call will create a directory `a` that contains directory `b`, etc.

#### Parameters

| Param | Type   | Description | Optional |
|:------|:------:|:------------|:--------:|
| path  | string | The path of directories that will be created. | ❌ |
| data  | string | The data you want to write into the file as string. | ✔️ |

#### Returns

| Name | Type   | Description | Provided On |
|:-----|:------:|:------------|:-----------:|
| success | boolean/nil | Whether or not said operation have succeeded. | Always |
| err | string | A string explaining what went wrong when executing said operation. | Failure Only |

----

### chroot(base)

***TODO***: Document this.

----
