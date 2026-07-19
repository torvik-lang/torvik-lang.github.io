---
title: Home
description: Torvik - a self-hosting, compiled, general-purpose programming language
---

# Torvik

**A self-hosting, compiled, general-purpose programming language.** Norse-inspired
keywords, native binaries through LLVM, automatic reference counting with no garbage
collector - and concurrency that is safe by construction.

The compiler (`torvc`) and the project tool (`rune`) are both written in Torvik itself.

## Install

Torvik runs on **Linux** and **Windows** (x86-64); macOS is planned once real Apple
hardware is available for testing. You'll need `clang` on your PATH - on Linux install
it from your package manager (`sudo apt install clang`, Solus: `sudo eopkg install
clang`); on Windows, LLVM/clang with a MinGW-w64 sysroot (the installer checks and
points you in the right direction if it's missing).

Linux:

```
curl -fsSL https://raw.githubusercontent.com/torvik-lang/torvik/main/linux/install.sh | sh
```

Windows (PowerShell):

```
iwr -useb https://raw.githubusercontent.com/torvik-lang/torvik/main/windows/install.ps1 | iex
```

Then:

```
rune new hello
cd hello
rune run
```

## Already on Torvik?

`rune` manages the toolchain from there - no need to re-run the installer:

```
rune update            # update to the latest release
rune update v1.3       # or pin: a release with 1.3.x 
rune version           # see what you're running
```

`rune update` fetches the release, verifies it, and swaps it in place;
`rune uninstall` removes the toolchain cleanly if you ever want it gone.

## Why Torvik

- **Native compilation** - Torvik emits LLVM IR linked with `clang`. Real executables,
  no virtual machine, no interpreter.
- **Self-hosting** - the language is proven on its hardest program: its own compiler.
- **No GC, no manual frees** - automatic reference counting gives deterministic memory
  management, and it stays **lock-free across threads** by design.
- **Concurrency without data races** - spawn tasks with `raven`, pass values over typed
  `bridge` channels. Values copy as they cross a thread, so tasks share no mutable
  state - a data race is not something you avoid, it is something you cannot write.
- **A small, sharp type system** - integer widths `i8` through `u128`, `f64`, `bool`,
  `str`, the `list`, `table`, and `bag` collections, `result` for explicit error
  handling, and `aett` enumerations with exhaustiveness-checked `when` matching.
- **Clean errors** - located, plain-language compile errors, plus a warnings system
  that found dead code in the compiler's own source the first time it ran.

## A taste

A worker reads jobs from a bridge until it closes - the raven carries everything it
needs in its claws:

```torvik
apply std;

df worker(jobs: bridge<i64>, results: bridge<i64>) -> void {
    whilst true {
        set r: result<i64> = try_recv(jobs);
        check is_err(r) { break; }
        fixed n: i64 = unwrap(r);
        send(results, n * n);
    }
}

df main() -> void {
    set jobs: bridge<i64> = bridge_new(8);
    set results: bridge<i64> = bridge_new(8);
    set h: task<void> = raven worker(jobs, results);

    set i: i64 = 1;
    whilst i <= 5 { send(jobs, i); i += 1; }
    bridge_close(jobs);

    set total: i64 = 0;
    set got: i64 = 0;
    whilst got < 5 { total += recv(results); got += 1; }
    join(h);

    echo!("sum of squares: {total}");
}
```

Take the [full tour](tour.html) for more of the language.

## Built with Torvik

> The Norns sit at the well beneath the world-tree, weaving the threads of fate.

[Vefna](vefna.html) is a static site generator written entirely in Torvik - it renders
pages in parallel on eight raven worker threads, and it wove the page you are reading
right now.

## Learn more

- [The Torvik Guide](https://github.com/torvik-lang/torvik/blob/main/docs/GUIDE.md) - full tutorial and language reference
- [Standard library](https://github.com/torvik-lang/torvik/blob/main/docs/STDLIB.md) - built-in function reference
- [Tooling](https://github.com/torvik-lang/torvik/blob/main/docs/TOOLING.md) - `torvc` and `rune`
- [Wiki](https://github.com/torvik-lang/torvik/wiki) - guides and FAQ
- [Releases](https://github.com/torvik-lang/torvik/releases) - changelogs and binaries
- [Discussions](https://github.com/torvik-lang/torvik/discussions) - questions, ideas, and show-and-tell
