---
title: Standard Library
description: The Torvik standard library - an optional layer above the core builtins, versioned independently of the compiler so a project can hold it steady while the toolchain moves on.
---

# The standard library

Torvik's core builtins are always available — `len`, `echo`, string and list operations,
files, and the rest. The **standard library** is the optional layer above them:

```torvik
apply std;              // the whole library
apply std::math;        // or just one part
```

As of Torvik v1.5.0 it lives in **[its own repository](https://github.com/torvik-lang/std)**
with its own version line. It still ships with the toolchain, and `rune update` keeps it
current — you don't install it separately.

---

## What's in it

| Module | What it gives you |
| --- | --- |
| `std::math` | `sign`, `isqrt`, `pow_checked` |
| `std::strings` | `strip_prefix`, `strip_suffix`, `is_digits`, `is_alpha`, `count_str` |
| `std::list` | `sort`, `reverse_list`, `index_of`, `contains_int`, `unique`, `join` |
| `std::path` | `path_base`, `path_dir`, path joining |
| `std::convert` | `to_hex`, `to_bin`, `from_hex`, `to_int` |
| `std::net` | A minimal opt-in HTTP layer, with binary-safe file serving |

The full reference, with every signature, lives in the repository:
**[docs/STDLIB.md](https://github.com/torvik-lang/std/blob/main/docs/STDLIB.md)**.

Applying one module pulls in only that module. Anything you don't use is dead-stripped out
of the finished binary, so `std::net` costs nothing in a program that never serves a request.

---

## Why it versions separately

Bundling a standard library with a compiler ties two schedules together that don't belong
together. A useful addition to `std::strings` shouldn't wait for a compiler release, and a
breaking change to a library function shouldn't force one.

So std carries its own semantic version. Your project can say what it needs:

```toml
[project]
name = "myapp"

std = "1.3.0"        # minimum standard library this project needs
```

If the installed library is older, the build stops with a clear message rather than failing
somewhere strange.

---

## Major versions are opt-in

`rune` will not move you across a **major** std version on its own. Within your current
major it keeps you current automatically; when a new major appears it tells you:

```
Heads up: standard library v2.0.0 is available - a NEW MAJOR (you're on v1.3.0).
  Staying on 1.x, since a major version can break existing code.
  To move up, either pin it in torvik.rune:
      std = 2.0.0
  or run:  rune update --std-major --yes
  Silence this notice:  rune update --silence-std-major
```

Two ways in, and both are deliberate acts:

```bash
rune update --std-major          # check only - reports, installs nothing
rune update --std-major --yes    # install it, and record the pin for you
```

With `--yes`, rune writes `std = <version>` into your manifest so the project and the
toolchain agree from then on. A pin in the manifest always wins — it's your project stating
what it needs, and that outranks the safety default.

---

## Going without

Some programs shouldn't have a standard library at all. A freestanding build has no
operating system to support one:

```toml
std = no_std
```

`apply std` is then refused at compile time. The same happens automatically under
`torvc --bare` — see [the systems chapter](/guide-systems).

---

## Contributing

std aims to stay small, predictable, and dependency-free. It is written entirely in Torvik
and uses only documented language features, so reading it is a reasonable way to learn the
language on something real.

[Contributing guide &rarr;](https://github.com/torvik-lang/std/blob/main/CONTRIBUTING.md)

---

## Learn more

- [Standard library repository](https://github.com/torvik-lang/std)
- [Full function reference](https://github.com/torvik-lang/std/blob/main/docs/STDLIB.md)
- [Guide: modules and projects](/guide-projects)
- [rune](/rune) — the tool that installs and updates it
