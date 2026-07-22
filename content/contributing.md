---
title: Contributing
description: How to contribute to Torvik, rune, and Vefna - reporting issues, submitting pull requests, and getting set up.
---

# Contributing

Torvik, rune, and Vefna are young, open-source (AGPL-3.0) projects, and community help
is genuinely valued. Each project keeps a `CONTRIBUTING.md` in its repository — the
authoritative guide — and this page is the short version with links straight to it.

All contributors are expected to follow the
[Code of Conduct](https://github.com/torvik-lang/torvik/blob/main/CODE_OF_CONDUCT.md).

## Ways to help

- **Report a bug** — open an issue with your version (`rune version` / `vefna version`),
  a minimal example that reproduces it, expected vs. actual behavior, and your platform.
- **Suggest a feature** — describe the problem it solves and why it fits the project's
  goals. For larger features, open an issue to discuss before building.
- **Improve the docs** — corrections and clarifications to the guides, the wiki, or this
  site are always welcome.
- **Send a pull request** — see the flow below.

## The general flow

1. **Open an issue first** (or comment on an existing one) to discuss the change —
   especially for new features or larger refactors. Small, obvious fixes can go straight
   to a PR.
2. **Fork** the repository and create a feature branch.
3. **Make your change** following the project's style and, for Torvik and rune, its
   self-hosting principles.
4. **Test thoroughly.** Every project ships a test suite; all tests must pass before and
   after your change.
5. **Update the docs** if behavior changed.
6. **Open the pull request** with a clear description of what and why. By contributing you
   agree your work is licensed under the project's **AGPL-3.0** license.

## By project

### Torvik

The language, compiler (`torvc`), standard library, and runtime.

- Code lives in `src/` (compiler), `src/std/` (standard library), and the runtime.
- Building the compiler uses maintainer tooling that isn't part of the public source. If
  you want to build Torvik directly, email **torviklang@gmail.com** and we'll send you the
  build scripts and, if needed, a `torvc` binary. For many changes (docs, standard-library
  code, examples) you can work against an installed toolchain without building the compiler
  from scratch.
- [Torvik CONTRIBUTING.md](https://github.com/torvik-lang/torvik/blob/main/CONTRIBUTING.md)
  · [Issues](https://github.com/torvik-lang/torvik/issues)

### rune

The project and toolchain manager, written in Torvik.

- Code lives in `src/rune.tv`; `src/diag.tv` is a vendored copy of Torvik's diagnostics
  module and is kept in sync with the Torvik repo rather than diverged.
- rune stays self-contained — it depends only on `torvc`, the standard library, and the
  vendored `diag`. For build tooling, email **torviklang@gmail.com**.
- [rune CONTRIBUTING.md](https://github.com/torvik-lang/rune/blob/main/CONTRIBUTING.md)
  · [Issues](https://github.com/torvik-lang/rune/issues)

### Vefna

The static site generator, written entirely in Torvik — and buildable with a public
toolchain.

1. Install Torvik (Vefna v1.1.0 builds with **Torvik v1.4.0**):

   ```
   curl -fsSL https://raw.githubusercontent.com/torvik-lang/torvik/main/linux/install.sh | sh
   ```

2. Build and test:

   ```
   rune build --final
   bash tests/run_tests.sh build/vefna                              # Linux
   powershell -ExecutionPolicy Bypass -File tests\run_tests.ps1     # Windows
   ```

   All tests must pass before and after your change. Check
   [ROADMAP.md](https://github.com/torvik-lang/vefna/blob/main/ROADMAP.md) — including the
   non-goals — before proposing a feature.

- [Vefna CONTRIBUTING.md](https://github.com/torvik-lang/vefna/blob/main/CONTRIBUTING.md)
  · [Issues](https://github.com/torvik-lang/vefna/issues)

## Questions

Not sure where something fits, or want to talk an idea through first? Reach out on the
[contact page](/contact) — email or social both work.
