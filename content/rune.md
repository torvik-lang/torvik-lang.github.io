---
title: Rune
description: rune - the package manager and toolchain manager for Torvik. Create, build, and run projects, and keep your installation up to date.
---

# Rune

**The project and toolchain manager for Torvik.** rune creates projects, builds and
runs them, and keeps your Torvik installation up to date. rune is itself written in
Torvik.

As of v1.4.0, rune lives in its own repository and is versioned and released
independently of Torvik, so it can ship fixes and features on its own cadence. It still
installs into the same place (`~/.torvik`) and manages Torvik exactly as before.

## Install

You normally get rune automatically when you install Torvik:

Linux:

```
curl -fsSL https://raw.githubusercontent.com/torvik-lang/torvik/main/linux/install.sh | sh
```

Windows (PowerShell):

```
iwr -useb https://raw.githubusercontent.com/torvik-lang/torvik/main/windows/install.ps1 | iex
```

The Torvik installer pulls the latest rune for you. To install or reinstall **only**
rune:

Linux:

```
curl -fsSL https://raw.githubusercontent.com/torvik-lang/rune/main/linux/install.sh | sh
```

Windows (PowerShell):

```
iwr -useb https://raw.githubusercontent.com/torvik-lang/rune/main/windows/install.ps1 | iex
```

## Commands

```
rune new <name>     Create a new project directory
rune init           Initialize a project in the current directory
rune build          Build the current project
rune run            Build and run the current project
rune run <file.tv>  Run a single file directly, forwarding any arguments
rune update [vX]    Update Torvik (torvc + std) and rune to the latest, or pin torvc to vX
rune self-update    Update just rune, from its own repo
rune uninstall      Remove the Torvik toolchain and rune (~/.torvik)
rune version        Show the rune, language, and std versions
```

A project is a `torvik.rune` manifest plus a `src/main.tv` entry point. The manifest can
declare a **minimum Torvik version** (and a minimum `std` version); `rune build` / `run`
refuses to build with an older toolchain and points you at `rune update`.

## Updating

```
rune update            # latest Torvik + rune
rune update v1.5       # pin: the newest 1.5.x toolchain
rune self-update       # just rune
```

Updates stay inside your **current major**. A new major can change or remove things your
code depends on, so rune reports one rather than installing it — while still pulling the
newest release within the major you're on:

```
Heads up: Torvik v2.0.0 is available - a NEW MAJOR (you're on v1.5.0).
  Staying on 1.x, since a major version can break existing code.
  When ready:  rune update v2 --yes
  Silence this notice:  rune update --silence-major
```

The standard library is gated the same way and has [its own repository](https://github.com/torvik-lang/std)
as of Torvik v1.5.0. Pin it with `std = "1.3.0"` in `torvik.rune`, or run
`rune update --std-major --yes` — which installs the new major *and* writes the pin for you.

For systems work, a `[build]` section in `torvik.rune` turns rune into a kernel build tool:
declare `target = bare`, an entry symbol, a link script, a boot stub and a `runner`, and
`rune build` / `rune run` do the rest. See [the systems chapter](/guide-systems).

`rune update` checks Torvik and rune **independently**: if only Torvik moved it updates
the toolchain (which also refreshes rune), if only rune moved it updates just rune, and if
both moved it updates both. Each update fetches the release, verifies it, and swaps it in
place.

## Uninstall

```
rune uninstall         # removes ~/.torvik (toolchain + rune)
```

rune follows semantic versioning on its own line, independent of Torvik's, and is
distributed as a prebuilt binary through the installers and the
[releases page](https://github.com/torvik-lang/rune/releases) — there's nothing to
compile.

## Learn more

- [rune repository](https://github.com/torvik-lang/rune) - source, releases, and changelog
- [Tooling guide](https://github.com/torvik-lang/torvik/blob/main/docs/TOOLING.md) - `torvc` and `rune` in depth
- [Back to Torvik](/)
