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

## Torvik — v1.4.0

The language and compiler.

**Install** &nbsp;
Linux: `curl -fsSL https://raw.githubusercontent.com/torvik-lang/torvik/main/linux/install.sh | sh`  
Windows (PowerShell): `iwr -useb https://raw.githubusercontent.com/torvik-lang/torvik/main/windows/install.ps1 | iex`

**Download** &nbsp;
Binary: [Linux (x86-64)](https://github.com/torvik-lang/torvik/releases/download/v1.4.0/torvc-linux-x86_64) &middot; [Windows (x86-64)](https://github.com/torvik-lang/torvik/releases/download/v1.4.0/torvc-windows-x86_64.exe)  
Source: [.zip](https://github.com/torvik-lang/torvik/archive/refs/tags/v1.4.0.zip) &middot; [.tar.gz](https://github.com/torvik-lang/torvik/archive/refs/tags/v1.4.0.tar.gz) &middot; [all assets](https://github.com/torvik-lang/torvik/releases/tag/v1.4.0)

**What's new**

- Optional (`^`) and variadic (`*`) function parameters, with call-site type checking
- Boolean chaining is limit-free — `&&` / `||` as values, `contains(...) == false`, and more
- `std::net` — an opt-in HTTP layer with binary-safe file serving
- `chr` and `fs_size` builtins; standard library v1.3.0
- `rune` decoupled into its own repository, versioned independently
- Fixes: exhaustive `when` false positive, boolean-literal argument IR, `bool` returns,
  single-character function names

[All Torvik releases &rarr;](https://github.com/torvik-lang/torvik/releases)

---

## rune — v1.4.0

The project and toolchain manager. Normally installed with Torvik; see the
[rune page](/rune) for the full command set.

**Install (rune only)** &nbsp;
Linux: `curl -fsSL https://raw.githubusercontent.com/torvik-lang/rune/main/linux/install.sh | sh`  
Windows (PowerShell): `iwr -useb https://raw.githubusercontent.com/torvik-lang/rune/main/windows/install.ps1 | iex`

**Download** &nbsp;
Binary: [Linux (x86-64)](https://github.com/torvik-lang/rune/releases/download/v1.4.0/rune-linux-x86_64) &middot; [Windows (x86-64)](https://github.com/torvik-lang/rune/releases/download/v1.4.0/rune-windows-x86_64.exe)  
Source: [.zip](https://github.com/torvik-lang/rune/archive/refs/tags/v1.4.0.zip) &middot; [.tar.gz](https://github.com/torvik-lang/rune/archive/refs/tags/v1.4.0.tar.gz) &middot; [all assets](https://github.com/torvik-lang/rune/releases/tag/v1.4.0)

**What's new**

- rune now lives in its own repository and versions independently of Torvik
- `rune update` checks Torvik and rune separately and updates whichever moved
- `rune self-update` updates just rune

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
