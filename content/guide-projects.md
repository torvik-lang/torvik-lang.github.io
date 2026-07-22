---
title: Guide 7 - Modules and projects
description: Split code across files with apply, use the Torvik standard library including std::net, and build and ship real projects with rune.
---

# 7. Modules, the standard library, and real projects

*[← 6. Concurrency](/guide-concurrency) · [Guide index](/guide) · Next: [Appendix: reference →](/guide-reference)*

Everything so far has lived in one file compiled by hand. Real projects outgrow both. This
chapter covers splitting code across files, the library that ships with Torvik, and the
project tool that ties it together.

## Modules: `apply`

`apply NAME;` brings another Torvik file's definitions into the current one.

Put shared code in its own file, `helpers.tv`:

```torvik
df shout(s: str) -> str {
    return str_concat(upper(s), "!");
}

df is_blank(s: str) -> bool {
    return len(trim(s)) == 0;
}
```

And use it from `main.tv`:

```torvik
apply helpers;

df main() -> void {
    echo!(shout("hello"));          // HELLO!
    echo!(is_blank("   "));         // true
}
```

`apply` lines go at the **top** of the file. The applied module's functions are then
available as if you'd written them there.

Split along **responsibility**, not size: parsing in one module, formatting in another,
business logic in a third. If you can't name what a module is *for*, it probably wants
splitting differently.

## The standard library

On top of the always-available core builtins (`len`, `str_concat`, `echo`, the collection
functions — everything used so far), Torvik ships an **opt-in standard library**.

```torvik
apply std;             // everything
```

Or take just what you need:

```torvik
apply std::math;
apply std::strings;
apply std::list;
apply std::path;
apply std::convert;
apply std::net;
```

The library is versioned **independently of the compiler** — it's currently 1.3.0 while the
compiler is 1.4.0. That's deliberate: a future breaking change can be gated behind a new
major library version you opt into in your project manifest, rather than being forced on you
by a compiler upgrade.

### `std::math`

```torvik
apply std::math;

echo!(pow(2, 10));       // 1024
echo!(abs(-5));          // 5
echo!(gcd(48, 36));      // 12
echo!(min(3, 7));
echo!(max(3, 7));
```

### `std::strings`

```torvik
apply std::strings;

echo!(join(split("a,b,c", ","), "-"));   // a-b-c
echo!(pad_left("7", 3, "0"));            // 007
echo!(repeat("ab", 3));                  // ababab
```

`join` is the counterpart to `split` — it's how you rebuild a line after processing its
fields.

### `std::list`

```torvik
apply std::list;

set xs: list<i64> = range(1, 6);   // [1, 2, 3, 4, 5]
echo!(sum(xs));                    // 15
```

### `std::path`

File path handling that works on both Linux and Windows — it accepts `/` and `\` on input:

```torvik
apply std::path;

echo!(path_base("dir/file.txt"));   // file.txt
echo!(path_dir("dir/file.txt"));    // dir
echo!(path_ext("dir/file.txt"));    // txt
echo!(path_join("dir", "file.txt"));
```

### `std::convert`

Base conversions and parsing that reports failure as a `result`:

```torvik
apply std::convert;

echo!(to_hex(255));       // ff
echo!(to_bin(5));         // 101
```

### `std::net` — a small HTTP layer

New in Torvik v1.4.0. `std::net` gives you enough to serve a static site or a small
localhost API. Its transport primitives are provided by the compiler and runtime and
**dead-strip out of programs that never call them**, so you pay nothing unless you use it.

The transport layer: `net_listen`, `net_accept`, `net_recv`, `net_send`, `net_send_file`,
`net_close`. On top of that, the module adds request parsing and response building:
`http_method`, `http_path`, `url_decode`, `path_is_safe`, `mime_type`, `send_text`,
`send_file_response`, `send_status`.

A minimal server:

```torvik
apply std::net;

df main() -> void {
    fixed lr: result<i64> = net_listen(8000);
    check is_err(lr) {
        echo!("could not listen: {err_msg(lr)}");
        return;
    }
    fixed server: i64 = unwrap(lr);
    echo!("serving on http://127.0.0.1:8000");

    whilst true {
        fixed client: i64 = net_accept(server);
        check client >= 0 {
            fixed req: str = net_recv(client);
            fixed path: str = http_path(req);

            check path_is_safe(path) {
                set _n: i64 = send_text(client, 200, "text/plain",
                                        fmt("you asked for {path}"));
            } fallback {
                set _f: i64 = send_status(client, 403, "Forbidden");
            }

            net_close(client);
        }
    }
}
```

Two things worth noting. `path_is_safe` rejects `..` traversal — check it before touching the
filesystem with a client-supplied path. And `net_send_file` streams a file's bytes
**verbatim**, so images, fonts, and archives serve intact even though Torvik strings are
NUL-terminated.

The model is deliberately simple and synchronous: accept a connection, read the request, send
one response, close. Sockets bind to `127.0.0.1` only, so a dev server is never accidentally
exposed to the network.

To opt out of the standard library entirely for a project, set `std = no_std` in its
manifest.

## Projects with `rune`

`rune` is the project tool. It creates projects, builds and runs them, and keeps your Torvik
installation current.

### Creating a project

```
rune new myproject
cd myproject
```

You get:

```
myproject/
  torvik.rune      the manifest
  src/
    main.tv        the entry point
```

The entry point is **always** `src/main.tv` — a convention, not something you configure.

### The manifest

```
[project]
name        = "myproject"
version     = "0.1.0"
description = "What this project does"
author      = "Your Name"
torvik      = "1.4.0"
std         = "1.3.0"

[runes]
# dependencies (coming with the Rune Library)
```

`torvik` and `std` declare **minimum versions**. If someone tries to build with an older
toolchain, they get a clean error pointing at `rune update` rather than a confusing failure
deep in the compile. Set them to what you actually use.

### Building and running

```
rune build          # compile to build/<name>
rune run            # compile and run
rune build --final  # optimized release build (-O3, stripped)
```

Use `rune run` while developing and `rune build --final` for anything you ship. The
difference in output size and speed is significant.

Other commands:

```
rune init           # make the current directory a project
rune list           # show project info
rune clean          # remove build/
rune version        # rune, language, and std versions
rune update         # update Torvik and rune
rune self-update    # update just rune
rune uninstall      # remove the toolchain
```

`rune update` pins too: `rune update v1.4` takes the newest 1.4.x.

### A project layout that scales

```
myproject/
  torvik.rune
  src/
    main.tv         entry point, argument handling
    config.tv       loading and validating settings
    parse.tv        input parsing
    report.tv       output formatting
  tests/
    run_tests.sh
```

with `main.tv` starting:

```torvik
apply std;
apply config;
apply parse;
apply report;

df main() -> void {
    // ...
}
```

Keep `main.tv` small — argument handling and orchestration. The real work belongs in modules
that can be reasoned about, and tested, on their own.

## Reading command-line arguments

```torvik
df main() -> void {
    fixed argc: i64 = args();

    check argc < 2 {
        echo!("usage: mytool <filename>");
        return;
    }

    fixed path: str = args_get(1);
    echo!("processing {path}");
}
```

`args()` gives the count and `args_get(i)` the argument at index `i`. Index 0 is the program
name, so real arguments start at 1.

A flag-handling shape that stays readable:

```torvik
df has_flag(flag: str) -> bool {
    fixed argc: i64 = args();
    set i: i64 = 1;
    whilst i < argc {
        fixed a: str = args_get(i);
        check a == flag { return true; }
        i += 1;
    }
    return false;
}

df main() -> void {
    fixed verbose: bool = has_flag("--verbose");
    check verbose { echo!("verbose mode"); }
}
```

## Working with files

```torvik
fixed text: result<str> = try_readfile("input.txt");
check is_err(text) {
    echo!("could not read input.txt: {err_msg(text)}");
    return;
}

each line in split(unwrap(text), "\n") {
    check len(trim(line)) > 0 {
        echo!(line);
    }
}
```

Other file operations include `writefile`, `appendfile`, `fs_exists`, `fs_size`, `fs_mkdir`,
`fs_remove`, `fs_copy`, `fs_is_dir`, and `dir_list`. The `try_` forms (`try_readfile`,
`try_writefile`, `try_appendfile`, `try_fs_copy`) return a `result` instead of halting — use
those whenever a missing or unreadable file is a realistic outcome, which is nearly always.

## A complete small tool

```torvik
apply std;

df count_lines(text: str) -> i64 {
    set n: i64 = 0;
    each line in split(text, "\n") {
        check len(trim(line)) > 0 { n += 1; }
    }
    return n;
}

df main() -> void {
    fixed argc: i64 = args();
    check argc < 2 {
        echo!("usage: linecount <file> [<file>...]");
        return;
    }

    set total: i64 = 0;
    set i: i64 = 1;

    whilst i < argc {
        fixed path: str = args_get(i);
        fixed content: result<str> = try_readfile(path);

        check is_err(content) {
            echo!("skipping {path}: {err_msg(content)}");
        } fallback {
            fixed n: i64 = count_lines(unwrap(content));
            echo!("{path}: {n}");
            total += n;
        }

        i += 1;
    }

    echo!("total: {total}");
}
```

Build it with `rune build --final` and you have a single binary that runs anywhere on that
platform with nothing installed.

## Things to try

1. Split one of your earlier programs into two files and wire them with `apply`.
2. Create a project with `rune new`, build it, and run it.
3. Set `torvik = "9.0.0"` in the manifest and try to build. Read the error, then fix it.
4. Write a tool that takes a filename and prints the longest line.
5. Add a `--count` flag that changes the output.
6. Run the `std::net` server and visit `http://127.0.0.1:8000/hello` in a browser.
7. Compare the size of `rune build` and `rune build --final` output.

---

*[← 6. Concurrency](/guide-concurrency) · [Guide index](/guide) · Next: [Appendix: reference →](/guide-reference)*
