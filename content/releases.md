---
title: Releases
description: Download Torvik, rune, and Vefna - current versions, source and binaries, and what's new in each latest release.
---

# Releases

Current releases of each project. Source archives download straight from GitHub; the
**binaries & installers** link opens the full GitHub release, where the platform builds
and other assets live. The one-line installers below are the recommended way to get set
up on Linux or Windows.

---

## Torvik — v1.5.0

The language and compiler.

**Install** &nbsp;
Linux: `curl -fsSL https://raw.githubusercontent.com/torvik-lang/torvik/main/linux/install.sh | sh`  
Windows (PowerShell): `iwr -useb https://raw.githubusercontent.com/torvik-lang/torvik/main/windows/install.ps1 | iex`

**Download** &nbsp;
Binary: [Linux (x86-64)](https://github.com/torvik-lang/torvik/releases/download/v1.5.0/torvc-linux-x86_64) &middot; [Windows (x86-64)](https://github.com/torvik-lang/torvik/releases/download/v1.5.0/torvc-windows-x86_64.exe)  
Source: [.zip](https://github.com/torvik-lang/torvik/archive/refs/tags/v1.5.0.zip) &middot; [.tar.gz](https://github.com/torvik-lang/torvik/archive/refs/tags/v1.5.0.tar.gz) &middot; [all assets](https://github.com/torvik-lang/torvik/releases/tag/v1.5.0)

**What's new**

**"The Forge" — systems and OS development.** Torvik can now build a freestanding image
that boots on bare hardware, with no operating system, no C library, and no runtime
beneath it. Everything new here is opt-in and `unsafe`-gated; the safe surface is unchanged.

- Raw pointers (`varda<T>`), bounds-checked fixed arrays (`[T; N]`), and `shape` value
  structs — a `packed shape` maps byte-for-byte onto a C struct or hardware descriptor
- `size_of` / `align_of`, volatile MMIO (`load_vol` / `store_vol`), inline assembly
  (`galdr`), and named x86_64 port intrinsics
- `torvc --bare` freestanding builds, with an allocator hook so heap types work bare —
  plus a complete [reference kernel and tutorial](/guide-systems)
- Float widths `f16`, `f32`, `f128` alongside `f64`, with true storage layout
- Hex, binary and underscored integer literals — `0xB8000`, `0b1010_0110`, `1_000_000`
- Run mode: `torvc file.tv` executes a file directly, no binary left behind
- The standard library moved to [its own repository](https://github.com/torvik-lang/std)
- **`&&` and `||` now short-circuit** — a correctness fix; they previously evaluated both
  sides, so `i < len(xs) && xs[i] == 1` could read out of bounds
- Many fixes, including several cases where a user mistake was reported as an internal
  compiler error

[All Torvik releases &rarr;](https://github.com/torvik-lang/torvik/releases)

---

## rune — v1.5.0

The project and toolchain manager. Normally installed with Torvik; see the
[rune page](/rune) for the full command set.

**Install (rune only)** &nbsp;
Linux: `curl -fsSL https://raw.githubusercontent.com/torvik-lang/rune/main/linux/install.sh | sh`  
Windows (PowerShell): `iwr -useb https://raw.githubusercontent.com/torvik-lang/rune/main/windows/install.ps1 | iex`

**Download** &nbsp;
Binary: [Linux (x86-64)](https://github.com/torvik-lang/rune/releases/download/v1.5.0/rune-linux-x86_64) &middot; [Windows (x86-64)](https://github.com/torvik-lang/rune/releases/download/v1.5.0/rune-windows-x86_64.exe)  
Source: [.zip](https://github.com/torvik-lang/rune/archive/refs/tags/v1.5.0.zip) &middot; [.tar.gz](https://github.com/torvik-lang/rune/archive/refs/tags/v1.5.0.tar.gz) &middot; [all assets](https://github.com/torvik-lang/rune/releases/tag/v1.5.0)

**What's new**

- `rune run <file.tv> [args...]` runs a single file directly, forwarding arguments
- A `[build]` manifest section — declare a freestanding target once and `rune build` /
  `rune run` build and launch a kernel
- **Major-version gating**: updates stay inside your current major and *report* a new one
  instead of installing it (`rune update v2 --yes` opts in)
- Standard-library gating, with `rune update --std-major --yes` writing the pin into your
  manifest for you
- rune now has its own test suite, so it can be checked without building the compiler

> **v1.4.0 is rune's first release in its own repository.** Earlier rune versions
> (v1.3.0 and before) shipped as part of Torvik — find those binaries under the matching
> [Torvik release assets](https://github.com/torvik-lang/torvik/releases). If you're on an
> older Torvik, you already have the rune that came with it; `rune update` moves you to the
> latest of both.

[All rune releases &rarr;](https://github.com/torvik-lang/rune/releases)

---

## Vefna — v1.1.0

The static site generator woven in Torvik.

**Install** &nbsp;
Linux: `curl -fsSL https://raw.githubusercontent.com/torvik-lang/vefna/main/linux/install.sh | sh`  
Windows (PowerShell): `iwr -useb https://raw.githubusercontent.com/torvik-lang/vefna/main/windows/install.ps1 | iex`

**Download** &nbsp;
Binary: [Linux (x86-64)](https://github.com/torvik-lang/vefna/releases/download/v1.1.0/vefna-linux-x86_64) &middot; [Windows (x86-64)](https://github.com/torvik-lang/vefna/releases/download/v1.1.0/vefna-windows-x86_64.exe)  
Source: [.zip](https://github.com/torvik-lang/vefna/archive/refs/tags/v1.1.0.zip) &middot; [.tar.gz](https://github.com/torvik-lang/vefna/archive/refs/tags/v1.1.0.tar.gz) &middot; [all assets](https://github.com/torvik-lang/vefna/releases/tag/v1.1.0)

**What's new**

- `vefna serve [port]` — a built-in preview server with rebuild-on-change
- `vefna watch` / `vefna build --watch` — rebuild continuously as sources change
- `--drafts` — `draft: true` pages are skipped unless you ask for them
- Built with Torvik v1.4.0

[All Vefna releases &rarr;](https://github.com/torvik-lang/vefna/releases)

---

Looking for older versions or full changelogs? Every project's **releases** page lists
each version with its notes and downloads.

---

## Support policy

Every major version is supported for **five years**, in three stages:

| Stage | Years | What it gets |
| --- | --- | --- |
| **Active** | 0 – 3 | New features, bug fixes, security fixes. Minor releases happen here. |
| **Maintenance** | 3 – 4 | Bug fixes and security fixes. No new features. |
| **Security** | 4 – 5 | Security fixes only. |

### Where things stand

| Line | Released | Stage | Active until | Maintenance until | End of life |
| --- | --- | --- | --- | --- | --- |
| **Torvik 1.x** | 4 Jul 2026 | **Active** | 4 Jul 2029 | 4 Jul 2030 | **4 Jul 2031** |
| **rune 1.x** | 4 Jul 2026 | **Active** | 4 Jul 2029 | 4 Jul 2030 | **4 Jul 2031** |
| **Vefna 1.x** | 4 Jul 2026 | **Active** | 4 Jul 2029 | 4 Jul 2030 | **4 Jul 2031** |

The standard library versions independently but tracks the toolchain line it ships
with: std 1.x is supported for as long as Torvik 1.x is.

**Nothing stops working at end of life** — a compiler that built your program last
year will build it next year. What ends is the flow of updates.

**You are never rushed onto a new major.** `rune update` deliberately keeps you inside
your current major and only tells you a new one exists, so a 1.x project keeps
receiving fixes for five years whether or not 2.0 has shipped. Upgrading stays a
decision you make, not one made for you.

Full policy: [SUPPORT.md](https://github.com/torvik-lang/torvik/blob/main/SUPPORT.md)
