---
title: Guide - Reference
description: Complete Torvik reference - every keyword and operator, the built-in function library, the memory model, and an honest list of what the language does not do yet.
---

# Appendix: full reference

*[← 7. Modules and projects](/guide-projects) · [Guide index](/guide)*

Everything in one place, for looking things up once you know the language.

## Keywords

| Keyword | Purpose |
|---|---|
| `df` | Define a function |
| `set` | Declare a mutable variable |
| `fixed` | Declare an immutable variable |
| `return` | Return from a function |
| `check` | Conditional (`if`) |
| `fallback` | Else branch of `check`, recovery block of `guard`, or default arm of `when` |
| `guard` | Require a condition, else run a fallback (early exit) |
| `whilst` | Condition loop (`while`) |
| `each` | Range or list loop |
| `in` | Separates the loop variable from its range in `each` |
| `break` | Exit the current loop |
| `continue` | Skip to the next loop iteration |
| `aett` | Declare a family of named values (an enumeration) |
| `when` | Pattern matching over an aett or integers |
| `raven` | Spawn a `df` function as a concurrent task |
| `task` | The type of a raven handle: `task<T>` |
| `echo` | Print without a trailing newline |
| `echo!` | Print with a trailing newline |
| `vow` | Assertion: abort with a message if a condition is false |
| `halt` | Print a message and exit immediately |
| `unsafe` | Prefix a declaration/assignment to opt into a rejected operation |
| `apply` | Bring another module's definitions into the file |
| `true` / `false` | Boolean literals |

## Operators

| Category | Operators | Notes |
|---|---|---|
| Arithmetic | `+` `-` `*` `/` `%` | Prefix `-` negates |
| Comparison | `==` `!=` `<` `>` `<=` `>=` | Yield `bool`; strings compare by content |
| Logical | `&&` `\|\|` `!` | Short-circuiting; usable as values |
| Bitwise | `&` `\|` `^` `~` `<<` `>>` | Integer types |
| Assignment | `=` `+=` `-=` `*=` `/=` `%=` | Compound forms update in place |
| Range | `..` `..+` | `..` exclusive, `..+` inclusive |
| Ternary | `?>` `!>` | `cond ?> a !> b`; only the taken branch runs |
| Weave | `~>` | `x ~> f ~> g` is `g(f(x))` |
| Membership | `<\|` | `item <\| collection` yields a `bool` |
| Variant | `::` | `Status::Active` reads an aett variant |
| When arm | `=>` | `pattern => statement-or-block` |

Arithmetic binds tighter than comparison, including with call and index operands:
`a + f(x) > b` reads as `(a + f(x)) > b`. Use parentheses whenever they clarify.

**Comparisons are fully chainable.** Any value — variable, literal, call result, list
element, `table_get`, or parenthesized expression — can sit on either side. Strings compare
by content; `<` `<=` `>` `>=` order them lexicographically. Comparing a string with a number
is a clean compile error.

**Boolean expressions are fully chainable (v1.4.0).** `&&` and `||` work as values, chain any
number of operands, and boolean-returning calls compare against literals:

```torvik
fixed ok: bool = a && b;
fixed all: bool = a && b && c;
check contains(line, "error") == false { ... }
```

## Types

| Type | Description |
|---|---|
| `i8` `i16` `i32` `i64` | Signed integers |
| `u8` `u16` `u32` `u64` | Unsigned integers |
| `i128` `u128` | Wide integers (heap-backed) |
| `f64` | 64-bit floating point |
| `bool` | `true` or `false` |
| `str` | Text |
| `list<T>` | Ordered, growable sequence |
| `table<K, V>` | Hash map |
| `bag<T>` | Set/multiset |
| `result<T>` | Success value or error |
| `task<T>` | Handle to a spawned task |
| `bridge<T>` | Typed channel between tasks |
| `void` | No return value |

## Built-in functions

Always available, no `apply` needed.

### Strings

| Function | Returns | Description |
|---|---|---|
| `len(s)` | `i64` | Length (also works on lists) |
| `str_concat(a, b, ...)` | `str` | Concatenate two or more strings |
| `substr(s, start, end)` | `str` | Substring, `end` exclusive |
| `char_at(s, i)` | `str` | Character at `i` as a 1-character string |
| `byte_at(s, i)` | `i64` | Raw byte value at `i` |
| `chr(code)` | `str` | 1-character string for a byte code |
| `trim(s)` / `triml(s)` / `trimr(s)` | `str` | Remove whitespace (both / left / right) |
| `upper(s)` / `lower(s)` | `str` | Case conversion |
| `replace(s, old, new)` | `str` | Replace occurrences |
| `contains(s, sub)` | `bool` | Whether `s` contains `sub` |
| `find(s, sub)` | `i64` | Index of first occurrence, or `-1` |
| `starts(s, prefix)` / `ends(s, suffix)` | `bool` | Prefix / suffix test |
| `split(s, sep)` | `list<str>` | Split into pieces |
| `fmt(template, ...)` | `str` | Build a string with interpolation |

### Conversions

| Function | Returns |
|---|---|
| `toint(s)` | `i64` |
| `tofloat(s)` | `f64` |
| `tostr(x)` | `str` |
| `try_toint(s)` | `result<i64>` |
| `try_tofloat(s)` | `result<f64>` |
| `typeof(x)` | `str` |

### Lists

`list_new()`, `push(xs, v)`, `xs[i]`, `len(xs)`, `list_insert(xs, i, v)`,
`list_remove(xs, i)`, `list_pop(xs)`

### Tables

`table_new()`, `table_set(t, k, v)`, `table_get(t, k)`, `table_has(t, k)`,
`table_del(t, k)`, `table_len(t)`, `table_keys(t)` (sorted)

### Bags

`bag_new()`, `bag_add(b, v)`, `bag_has(b, v)`, `bag_remove(b, v)`, `bag_len(b)`

### Results

`ok(v)`, `err(msg)`, `err(code, msg)`, `is_ok(r)`, `is_err(r)`, `unwrap(r)`,
`unwrap_or(r, default)`, `err_msg(r)`, `err_code(r)`

### Files and system

`readfile(p)`, `writefile(p, s)`, `appendfile(p, s)`, `fs_exists(p)`, `fs_size(p)`,
`fs_mkdir(p)`, `fs_remove(p)`, `fs_copy(a, b)`, `fs_is_dir(p)`, `fs_mtime(p)`,
`dir_list(p)`, `args()`, `args_get(i)`, `exit(code)`, `sleep(ms)`

Result-returning forms: `try_readfile`, `try_writefile`, `try_appendfile`, `try_fs_copy`.

### Concurrency

`raven f(...)`, `join(h)`, `bridge_new(cap)`, `send(ch, v)`, `recv(ch)`, `try_recv(ch)`,
`bridge_close(ch)`

## The memory model

Torvik manages memory with **automatic reference counting (ARC)**. Heap values — strings,
lists, tables, bags, 128-bit integers — carry a count of how many references point at them.
When the last one goes away, the memory is released immediately.

There is no garbage collector and no manual `free`.

What that gives you:

- **Deterministic cleanup.** Memory is released at a predictable point, not whenever a
  collector decides to run. No pauses.
- **Leak-free for supported patterns.** Ordinary code that builds and discards strings and
  collections releases memory deterministically.
- **Clean out-of-memory behaviour.** If an allocation can't be satisfied, the program panics
  cleanly rather than corrupting state.
- **No reference cycles.** Reference counting can't reclaim cycles — but Torvik has no
  construct that can create one. (That would require nested mutable containers, which arrive
  in a later version alongside a cycle strategy.)
- **Lock-free across threads.** Concurrency copies values as they cross thread boundaries, so
  ordinary refcounts never need atomics. The single exception is a bridge's own refcount — a
  bridge is deliberately shared, so that one count is atomic. Nothing else in the language is.

You don't manage any of this. It's worth understanding because it explains the
[self-contained rule](/guide-concurrency) for tasks, and why Torvik has no GC pauses.

## Compile warnings

| Category | Triggered by |
|---|---|
| `unused_variable` | A binding that's never read |
| `unreachable_code` | A statement after `return`/`break`/`continue`/`halt`/`exit` |
| `unused_result` | A bare statement call of a non-void function |
| `deprecated` | A builtin scheduled for removal |

Prefix a name with `_` to mark a discard as deliberate. Control from inside a file with
`!@NO_WARN;` or `!@ALLOW[category];` at the top. Suppress from the command line with
`torvc --no-warn`.

## Compiler and project tools

```
torvc file.tv -o output      compile
torvc file.tv -o out --final optimized build (-O3, stripped)
torvc --no-warn              suppress warnings
torvc version                version info

rune new <name>              create a project
rune init                    make the current directory a project
rune build                   build to build/<name>
rune build --final           optimized release build
rune run                     build and run
rune list                    project info
rune clean                   remove build/
rune version                 rune, language, and std versions
rune update [vX]             update Torvik and rune (optionally pinned)
rune self-update             update just rune
rune uninstall               remove the toolchain
```

## What Torvik does not do yet

An honest list. Every one of these is a clean compile error or a documented narrow case —
never a silent wrong answer.

**Not in the language yet:**

- **Structs (`shape`)** — grouping named fields into one value.
- **A `pub` visibility keyword.**
- **`f32`** and a dedicated **`char`** type (character literals are one-character strings
  today).
- **Fixed-size arrays** (`[T; N]`).
- **Systems/OS primitives** — inline assembly, volatile access, raw pointer operations,
  packed structs. Torvik is a general-purpose compiled language; these are out of scope.

**Narrow cases:**

- A conversion result used directly inside a larger arithmetic expression needs binding
  first: write `set n: i64 = toint("7"); echo!(n + 1);`.
- Collections and `result` values don't cross a spawn or a bridge yet — pass scalars and
  strings and rebuild structure inside the task.
- `bridge_new` needs a capacity of at least 1; there are no unbuffered rendezvous channels.
- There's no `select` over multiple bridges.

**Platforms:** Linux and Windows (x86-64). macOS is planned once Apple hardware is available
for credible testing.

## Where to go next

You've been through the whole language. The most useful next step is to build something —
a tool you'd actually use. A line counter, a log filter, a small static site generator, a
localhost API.

- [Rune](/rune) — the project and toolchain manager in detail
- [Vefna](/vefna) — a static site generator written entirely in Torvik, and a substantial
  real-world Torvik program to read
- [Releases](/releases) — downloads and changelogs
- [Contributing](/contributing) — how to help
- [Contact](/contact) — questions and feedback

---

*[← 7. Modules and projects](/guide-projects) · [Guide index](/guide)*
