# `std::filesystem` in C++17

## Table of Contents

- [Introduction](#introduction)
- [Rationale](#rationale)
- [Required Header and Namespace](#required-header-and-namespace)
- [Compiler and Library Support](#compiler-and-library-support)
- [Core Types](#core-types)
- [The `path` Class](#the-path-class)
- [Path Inspection](#path-inspection)
- [Composing Paths](#composing-paths)
- [Absolute, Relative, and Normalized Paths](#absolute-relative-and-normalized-paths)
- [Status Checks](#status-checks)
- [Iterating with `directory_iterator`](#iterating-with-directory_iterator)
- [Iterating with `recursive_directory_iterator`](#iterating-with-recursive_directory_iterator)
- [Working with `directory_entry`](#working-with-directory_entry)
- [Creating, Copying, Renaming, and Removing](#creating-copying-renaming-and-removing)
- [File Size, Disk Space, and Time Stamps](#file-size-disk-space-and-time-stamps)
- [Symbolic Links](#symbolic-links)
- [Permissions](#permissions)
- [Exception and `error_code` Handling](#exception-and-error_code-handling)
- [Complete Example](#complete-example)
- [Boost Comparison](#boost-comparison)
- [Common Errors and Security Notes](#common-errors-and-security-notes)
- [Exercises](#exercises)
- [Sources and Technical Accuracy](#sources-and-technical-accuracy)
- [Conclusion](#conclusion)
- [Contributors](#contributors)

---

## Introduction

`std::filesystem` is the C++17 library for working with paths, files, directories, timestamps, permissions, symbolic links, and available storage. It replaces a large amount of platform-specific code with a portable API that is good enough for most everyday filesystem tasks.

Before C++17, developers often used operating-system APIs directly or depended on Boost.Filesystem. C++17 standardized that functionality and kept the overall design close to the older Boost interface, which made adoption easier.

Typical uses include:

- inspecting and composing paths
- checking whether a file or directory exists
- iterating through a directory tree
- creating directories and copying files
- querying file size and free disk space
- reading and changing permissions
- handling symbolic links safely

---

## Rationale

Filesystem code is common, but it is also one of the least portable parts of a program. Path separators differ across platforms, some APIs treat paths as text while others treat them as structured objects, and error handling often depends on the underlying operating system.

`std::filesystem` exists to solve a few concrete problems:

- path syntax is handled portably instead of manually concatenating strings
- file and directory queries use standard types and standard return values
- errors can be handled with exceptions or `std::error_code`
- common operations such as copying, removing, and iterating are available in one place

This makes code easier to read, easier to test, and less dependent on platform-specific details.

---

## Required Header and Namespace

Include the `<filesystem>` header and use the `std::filesystem` namespace.

```cpp
#include <filesystem>

namespace fs = std::filesystem;
```

A namespace alias is common because the fully qualified names are long and appear often in real code.

---

## Compiler and Library Support

C++17 standardized `std::filesystem`, but toolchain support arrived in stages.

- GCC 8 and newer generally provide it in libstdc++
- Clang support depends on the standard library in use, usually libc++ or libstdc++
- MSVC supports it in Visual Studio 2017 version 15.7 and newer

There is one important historical caveat: some older GCC and Clang setups require linking with `-lstdc++fs` even when compiling with `-std=c++17`. That was necessary on implementations where the filesystem library lived in a separate static library.

Examples:

```bash
g++ -std=c++17 main.cpp -lstdc++fs
clang++ -std=c++17 main.cpp -lstdc++fs
```

On modern toolchains, that flag is usually no longer needed.

---

## Core Types

The library is built around a small set of core types.

- `fs::path`: an object representing a filesystem path
- `fs::directory_entry`: a cached view of a directory item
- `fs::directory_iterator`: iterates one directory level
- `fs::recursive_directory_iterator`: iterates recursively
- `fs::file_status`: describes the type and permissions of a file
- `fs::space_info`: describes disk capacity, free space, and available space
- `fs::file_time_type`: represents filesystem timestamps

There are also supporting enums and option types:

- `fs::file_type`
- `fs::perms`
- `fs::copy_options`
- `fs::perm_options`
- `fs::directory_options`

---

## The `path` Class

`fs::path` is the central abstraction. It stores a path in a platform-aware format and provides structured access to its parts.

A path is not just a string. It understands concepts such as root name, root directory, parent path, filename, stem, and extension.

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path p = "C:/projects/demo/main.cpp";

    std::cout << "path: " << p << '\n';
    std::cout << "generic_string: " << p.generic_string() << '\n';
    std::cout << "native: " << p.native() << '\n';
}
```

`path` is designed to work correctly with platform separators and encoding rules. On Windows, it handles wide and narrow path conversions more carefully than raw string concatenation.

---

## Path Inspection

A path can be inspected component by component.

```cpp
fs::path p = "/home/user/archive.tar.gz";

std::cout << "root_name: " << p.root_name() << '\n';
std::cout << "root_directory: " << p.root_directory() << '\n';
std::cout << "root_path: " << p.root_path() << '\n';
std::cout << "relative_path: " << p.relative_path() << '\n';
std::cout << "parent_path: " << p.parent_path() << '\n';
std::cout << "filename: " << p.filename() << '\n';
std::cout << "stem: " << p.stem() << '\n';
std::cout << "extension: " << p.extension() << '\n';
```

For `/home/user/archive.tar.gz`, the result is conceptually:

- `root_name`: empty on POSIX
- `root_directory`: `/`
- `root_path`: `/`
- `relative_path`: `home/user/archive.tar.gz`
- `parent_path`: `/home/user`
- `filename`: `archive.tar.gz`
- `stem`: `archive.tar`
- `extension`: `.gz`

Useful inspection members and functions include:

- `empty()`
- `has_root_path()`
- `has_root_name()`
- `has_root_directory()`
- `has_relative_path()`
- `has_parent_path()`
- `has_filename()`
- `has_stem()`
- `has_extension()`
- `is_absolute()`
- `is_relative()`

You can also iterate over a path as a sequence of components:

```cpp
for (const auto& part : p) {
    std::cout << '[' << part << "]\n";
}
```

That is useful for diagnostics and for writing path-aware logic without manual string parsing.

---

## Composing Paths

Use path-aware operations to build paths instead of concatenating raw strings.

```cpp
fs::path base = "/var";
fs::path logs = base / "log" / "app";

std::cout << logs << '\n';
```

Important operators and members:

- `/` and `/=` insert a preferred separator when needed
- `append()` behaves like `/=`
- `+=` concatenates text directly and does not insert a separator

```cpp
fs::path a = "/tmp";
a /= "cache";

fs::path b = "/tmp";
b += "cache";

std::cout << a << '\n'; // /tmp/cache
std::cout << b << '\n'; // /tmpcache
```

Use `/=` for path segments. Use `+=` only when you intentionally want raw concatenation.

The class also supports parent manipulation:

- `remove_filename()`
- `replace_filename()`
- `replace_extension()`
- `lexically_normal()`
- `lexically_relative()`
- `lexically_proximate()`

---

## Absolute, Relative, and Normalized Paths

`std::filesystem` includes several ways to turn a path into another form.

### `absolute`

Returns an absolute path by combining a relative path with the current working directory.

```cpp
fs::path p = "data/input.txt";
fs::path abs = fs::absolute(p);
```

### `relative_path`, `relative`, and `proximate`

These functions help convert between base paths and target paths.

```cpp
fs::path base = "/home/user/project";
fs::path target = "/home/user/project/src/main.cpp";

std::cout << target.lexically_relative(base) << '\n';
```

`lexically_relative()` works without consulting the actual filesystem. That makes it useful for pure path manipulation.

### `canonical` and `weakly_canonical`

- `canonical()` resolves symlinks and requires the path to exist
- `weakly_canonical()` is more forgiving and can normalize paths that partially exist

### `lexically_normal`

This removes redundant `.` and `..` segments where possible.

```cpp
fs::path p = "/a/./b/../c/";
std::cout << p.lexically_normal() << '\n'; // /a/c/
```

Lexical normalization is not the same as resolving actual filesystem links. It only rewrites the path text according to path rules.

---

## Status Checks

A large portion of filesystem code is status checking. The standard library gives you both quick predicates and lower-level status access.

Common predicates:

- `exists(path)`
- `is_directory(path)`
- `is_regular_file(path)`
- `is_symlink(path)`
- `is_block_file(path)`
- `is_character_file(path)`
- `is_fifo(path)`
- `is_socket(path)`
- `is_other(path)`
- `is_empty(path)`

Example:

```cpp
fs::path p = "/etc/passwd";

std::cout << std::boolalpha;
std::cout << "exists: " << fs::exists(p) << '\n';
std::cout << "regular file: " << fs::is_regular_file(p) << '\n';
std::cout << "directory: " << fs::is_directory(p) << '\n';
```

If you need more detail, use `status()` or `symlink_status()`:

```cpp
fs::file_status st = fs::status(p);
std::cout << static_cast<int>(st.type()) << '\n';
```

The difference matters for symbolic links:

- `status()` follows the link target
- `symlink_status()` reports the link itself

That distinction is important for security-sensitive code and for tools that need to display links rather than their targets.

---

## Iterating with `directory_iterator`

`directory_iterator` iterates over the immediate children of a directory.

```cpp
fs::path dir = ".";

for (const fs::directory_entry& entry : fs::directory_iterator(dir)) {
    std::cout << entry.path() << '\n';
}
```

Properties of this iterator:

- it does not recurse into subdirectories
- `.` and `..` are not produced
- iteration order is unspecified
- it can be constructed from a path, a `directory_options` flag set, or with error reporting overloads

This iterator is appropriate when you want to list one folder, count files, or perform a shallow scan.

---

## Iterating with `recursive_directory_iterator`

`recursive_directory_iterator` walks a directory tree depth-first.

```cpp
for (const auto& entry : fs::recursive_directory_iterator(".")) {
    std::cout << entry.path() << '\n';
}
```

Useful member functions include:

- `depth()`
- `recursion_pending()`
- `disable_recursion_pending()`
- `pop()`

Example of skipping a directory named `build`:

```cpp
for (auto it = fs::recursive_directory_iterator("."); it != fs::recursive_directory_iterator(); ++it) {
    if (it->path().filename() == "build") {
        it.disable_recursion_pending();
    }
    std::cout << it->path() << '\n';
}
```

That gives you finer control than a plain recursive function.

---

## Working with `directory_entry`

`directory_entry` stores a path and often caches metadata that can be queried efficiently.

```cpp
for (const auto& entry : fs::directory_iterator(".")) {
    std::cout << entry.path() << '\n';
    std::cout << "  file_size query is for regular files only\n";
}
```

Common operations:

- `path()`
- `status()`
- `symlink_status()`
- `exists()`
- `is_directory()`
- `is_regular_file()`
- `is_symlink()`
- `refresh()`

The cached status makes repeated queries cheaper in many implementations, especially when you inspect the same entry several times.

---

## Creating, Copying, Renaming, and Removing

The library supports common filesystem mutations.

### Creating directories

```cpp
fs::create_directory("logs");
fs::create_directories("logs/2025/08");
```

- `create_directory()` creates one directory level
- `create_directories()` creates parents as needed

### Copying

```cpp
fs::copy_file("input.txt", "backup.txt", fs::copy_options::overwrite_existing);
fs::copy("src_dir", "dst_dir", fs::copy_options::recursive);
```

`copy_file()` is for files. `copy()` is more general and can copy directory trees.

### Renaming

```cpp
fs::rename("old_name.txt", "new_name.txt");
```

`rename()` is often the best choice for atomic replacement on the same filesystem, subject to platform rules.

### Removing

```cpp
fs::remove("temporary.txt");
fs::remove_all("old_cache");
```

- `remove()` removes one file or empty directory
- `remove_all()` removes a path recursively and returns the count of removed items

Be careful with `remove_all()`. It is powerful and dangerous if the path is wrong.

---

## File Size, Disk Space, and Time Stamps

### File size

`file_size()` works only for regular files.

```cpp
auto size = fs::file_size("report.txt");
std::cout << size << '\n';
```

For directories or special files, use `is_regular_file()` first.

### Disk space

`space()` returns a `space_info` struct with three fields:

- `capacity`
- `free`
- `available`

```cpp
fs::space_info info = fs::space(".");
std::cout << "capacity: " << info.capacity << '\n';
std::cout << "free: " << info.free << '\n';
std::cout << "available: " << info.available << '\n';
```

### Last write time

`last_write_time()` reads or updates the file timestamp.

```cpp
auto t = fs::last_write_time("report.txt");
```

The exact conversion to human-readable time depends on the standard library version because `fs::file_time_type` is not the same as `std::chrono::system_clock::time_point` on every implementation. In real code, conversion is usually handled through a helper function.

---

## Symbolic Links

The library exposes symbolic link support through both queries and operations.

### Checking links

```cpp
if (fs::is_symlink("config-link")) {
    std::cout << "link\n";
}
```

### Creating links

```cpp
fs::create_symlink("target.txt", "link.txt");
fs::create_directory_symlink("target_dir", "dir_link");
```

### Reading link targets

```cpp
fs::path target = fs::read_symlink("link.txt");
```

Use `symlink_status()` when you need to inspect the link itself rather than the file it points to.

Symbolic links deserve extra care in security-sensitive code because the target can be changed after validation if you are not careful about the sequence of operations.

---

## Permissions

Permissions are represented by `fs::perms`.

```cpp
fs::perms p = fs::status("script.sh").permissions();
```

You can read and modify permissions with `permissions()`.

```cpp
fs::permissions("script.sh", fs::perms::owner_exec, fs::perm_options::add);
```

Common operations are:

- `perm_options::replace`
- `perm_options::add`
- `perm_options::remove`
- `perm_options::nofollow`

Permission APIs behave differently across platforms, especially between POSIX systems and Windows. Do not assume the same permission bits always mean the same thing everywhere.

---

## Exception and `error_code` Handling

Most filesystem functions have two overload families.

### Throwing overloads

These throw `fs::filesystem_error` on failure.

```cpp
try {
    fs::copy_file("a.txt", "b.txt");
} catch (const fs::filesystem_error& e) {
    std::cerr << e.what() << '\n';
}
```

### Non-throwing overloads

These take `std::error_code` and report errors without exceptions.

```cpp
std::error_code ec;
fs::copy_file("a.txt", "b.txt", fs::copy_options::none, ec);
if (ec) {
    std::cerr << ec.message() << '\n';
}
```

Use `error_code` overloads when failure is expected and part of normal control flow, such as scanning a directory tree that may contain permission-denied entries.

Use throwing overloads when failure should abort the operation and unwind to a higher level.

---

## Complete Example

The following example demonstrates the most common pieces together: path inspection, directory traversal, file size, timestamps, copying, renaming, and error handling.

```cpp
#include <filesystem>
#include <iostream>
#include <string>

namespace fs = std::filesystem;

void PrintEntry(const fs::directory_entry& entry)
{
    std::cout << entry.path() << '\n';

    std::error_code ec;
    if (entry.is_regular_file(ec)) {
        auto size = entry.file_size(ec);
        if (!ec) {
            std::cout << "  size: " << size << '\n';
        }
    }

    if (entry.is_symlink(ec) && !ec) {
        std::cout << "  symlink target: " << fs::read_symlink(entry, ec) << '\n';
    }
}

int main(int argc, char* argv[])
{
    fs::path root = (argc > 1) ? fs::path(argv[1]) : fs::current_path();

    std::cout << "root: " << fs::absolute(root) << '\n';
    std::cout << "normalized: " << root.lexically_normal() << '\n';

    std::error_code ec;
    if (!fs::exists(root, ec)) {
        std::cerr << "path does not exist: " << root << '\n';
        return 1;
    }

    if (fs::is_regular_file(root, ec)) {
        std::cout << "single file\n";
        std::cout << "size: " << fs::file_size(root, ec) << '\n';
        if (ec) {
            std::cerr << ec.message() << '\n';
            return 1;
        }
        return 0;
    }

    if (!fs::is_directory(root, ec) || ec) {
        std::cerr << "not a directory\n";
        if (ec) {
            std::cerr << ec.message() << '\n';
        }
        return 1;
    }

    for (const auto& entry : fs::directory_iterator(root, ec)) {
        if (ec) {
            std::cerr << ec.message() << '\n';
            break;
        }
        PrintEntry(entry);
    }

    fs::path backup_dir = root / "backup";
    fs::create_directories(backup_dir, ec);
    if (ec) {
        std::cerr << "create_directories failed: " << ec.message() << '\n';
        return 1;
    }

    fs::path sample = backup_dir / "sample.txt";
    fs::path copied = backup_dir / "sample-copy.txt";

    if (fs::exists(sample) && fs::is_regular_file(sample)) {
        fs::copy_file(sample, copied, fs::copy_options::overwrite_existing, ec);
        if (ec) {
            std::cerr << "copy_file failed: " << ec.message() << '\n';
            return 1;
        }

        fs::rename(copied, backup_dir / "renamed-sample.txt", ec);
        if (ec) {
            std::cerr << "rename failed: " << ec.message() << '\n';
            return 1;
        }
    }

    fs::space_info info = fs::space(root, ec);
    if (!ec) {
        std::cout << "capacity: " << info.capacity << '\n';
        std::cout << "free: " << info.free << '\n';
        std::cout << "available: " << info.available << '\n';
    }

    return 0;
}
```

This example is intentionally conservative:

- it validates the root path before using it
- it uses `error_code` overloads for operations that may fail during normal scanning
- it treats files and directories differently
- it avoids assuming that a timestamp conversion or permission mapping is identical across platforms

---

## Boost Comparison

`std::filesystem` is heavily inspired by Boost.Filesystem. In practice, the APIs are similar, and much Boost-era code can be migrated with small edits.

### Similarities

- `path`-centric design
- iterator types for directory traversal
- operations for copy, rename, remove, and status queries
- throwing and non-throwing error handling styles

### Differences

- namespace changes from `boost::filesystem` to `std::filesystem`
- the standard library version is the long-term target in new code
- some Boost releases historically offered broader compatibility with very old compilers
- edge-case behavior and formatting details can differ slightly between implementations

### When Boost still matters

Boost.Filesystem may still be useful when you need to support older toolchains that do not have a complete C++17 library implementation. For new code on modern compilers, `std::filesystem` is usually the better default.

---

## Common Errors and Security Notes

1. **Using string concatenation for paths**: this often breaks on platform separators or loses path structure. Prefer `path / other`.
2. **Forgetting that `file_size()` is for regular files**: calling it on a directory or special file can fail.
3. **Assuming iteration order is stable**: directory iteration order is unspecified.
4. **Ignoring symlink behavior**: `status()` and `symlink_status()` are not interchangeable.
5. **Deleting the wrong tree**: `remove_all()` is recursive and should be guarded carefully.
6. **Trusting user-supplied paths blindly**: validate roots, normalize carefully, and avoid path traversal when processing external input.
7. **Confusing lexical normalization with real resolution**: `lexically_normal()` does not resolve links or require the path to exist.
8. **Assuming permissions are identical across platforms**: permission bits are not a perfect cross-platform abstraction.
9. **Relying on `canonical()` for non-existent paths**: it requires that the target exist; use `weakly_canonical()` when appropriate.
10. **Ignoring time and race conditions**: a file can change between a status check and a later operation. Do not treat one check as a guarantee.

Security-wise, the biggest mistake is to validate a path as text and then perform a separate file operation without considering symlink substitution or concurrent changes. When the path comes from an untrusted source, keep the allowed base directory narrow and re-check at the point of use.

---

## Exercises

1. Write a program that lists all regular files in a directory and prints their sizes in bytes.
2. Extend it to recurse into subdirectories using `recursive_directory_iterator`.
3. Add filtering so only `.cpp` and `.h` files are printed.
4. Create a function that copies one directory tree to another using `copy()` and `copy_options::recursive`.
5. Implement a safe delete helper that refuses to call `remove_all()` unless the path is inside a known workspace directory.
6. Compare the output of `lexically_normal()`, `absolute()`, and `canonical()` on a path that contains `.` and `..` segments.

---

## Sources and Technical Accuracy

This tutorial follows the C++17 standard library model for `std::filesystem` and the standard naming used by implementations such as libstdc++, libc++, and MSVC STL.

Good references for verification and further reading are:

- cppreference: `std::filesystem`
- the C++17 standard library specification for filesystem
- Boost.Filesystem documentation for migration context
- compiler vendor documentation for toolchain-specific linking notes

Implementation details such as timestamp conversions, permission mappings, and some platform-specific edge cases can vary slightly across standard library vendors. The API surface and the core semantics described here are the stable parts.

---

## Conclusion

`std::filesystem` is one of the most useful C++17 additions because it solves a real portability problem with a standard API. It gives you structured path handling, directory traversal, status queries, file operations, permissions, timestamps, and disk-space information without forcing you into raw OS-specific code.

For most modern C++ code, it should be the default way to work with the filesystem.

---

## Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |  |  |  |
