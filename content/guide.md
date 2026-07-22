---
title: Language Guide
description: Learn the Torvik programming language from scratch - a complete, detailed tutorial that takes you from no programming experience at all to writing real, concurrent, production Torvik programs.
---

# The Torvik Language Guide

This is the complete guide to Torvik. It starts from nothing — no programming experience
assumed — and goes all the way through the language, including memory, concurrency, and
shipping real projects.

You don't need to have written code before. You don't need to know another language. Every
concept is introduced before it's used, and every example is a complete, runnable program or
a fragment you can drop into one.

If you *have* programmed before, skim [Chapter 1](/guide-basics) for the syntax and jump to
whatever you need — the chapters stand on their own once you know how a Torvik program is
laid out.

## What is Torvik?

Torvik is a **compiled, statically-typed, general-purpose programming language**.

Piece by piece:

- **Compiled** — you write a text file, run it through the Torvik compiler (`torvc`), and get
  a native executable: a real program your operating system runs directly. There's no
  interpreter or virtual machine sitting underneath it at run time.
- **Statically typed** — every value has a type (a whole number, a piece of text, a list of
  things), and the compiler knows all of them before your program ever runs. If you try to
  add a number to a word, you're told at compile time, not by a crash in front of a user.
- **General-purpose** — it isn't aimed at one niche. Command-line tools, servers, text
  processing, build tooling, small web services.

Two more things make it unusual:

- **No garbage collector.** Memory is reclaimed automatically using reference counting, which
  means cleanup is *deterministic* — it happens at a predictable moment, not whenever a
  collector decides to run. You never write `free`, and you never get a surprise pause.
- **It is written in itself.** The Torvik compiler is a Torvik program. It compiles its own
  source code into the next version of itself, and the result is byte-for-byte identical
  every time. A language that can build itself is a language that has to be genuinely
  complete.

The keywords are Norse-inspired — `df` to define a function, `check` for a condition,
`whilst` for a loop, `raven` to spawn a concurrent task. They read as words, not
abbreviations, and they're consistent: there is one way to say each thing.

## How to use this guide

Work through it in order the first time. Each chapter builds on the one before, and later
chapters assume the vocabulary of earlier ones.

Type the examples out rather than copying them. Programming is a motor skill as much as an
intellectual one, and the mistakes you make while typing are where most of the learning
happens. When something doesn't compile, read the error — Torvik's errors point at the exact
character and say what it expected.

Every chapter ends with things to try. They're worth doing.

## The chapters

**[1. First steps](/guide-basics)**
Installing Torvik, your first program, and the raw material of all code: comments, variables,
types, operators, and printing. By the end you can write a program that stores values, does
arithmetic, and shows results.

**[2. Making decisions and repeating work](/guide-flow)**
Branching with `check` and `fallback`, the `?>` ternary, early exits with `guard`, loops with
`whilst` and `each`, and pattern matching with `aett` and `when`.

**[3. Functions](/guide-functions)**
Defining your own operations with `df`, parameters and return values, optional (`^`) and
variadic (`*`) parameters, recursion, scope, and the weave operator `~>`.

**[4. Collections and text](/guide-collections)**
Lists, tables, and bags — storing many values at once — plus the full set of string
operations and the membership operator `<|`.

**[5. Errors, safety, and the compiler on your side](/guide-errors)**
Handling failure with `result`, `ok`, and `err`; asserting invariants with `vow` and `halt`;
reading compile warnings; and the narrow, explicit `unsafe` prefix.

**[6. Concurrency: ravens and bridges](/guide-concurrency)**
Doing several things at once. Spawning tasks with `raven`, passing values over typed `bridge`
channels, and why Torvik gives you no way to write a data race.

**[7. Modules, the standard library, and real projects](/guide-projects)**
Splitting code across files with `apply`, using the standard library, and building and
shipping projects with `rune`.

**[Appendix: full reference](/guide-reference)**
Every keyword and operator in one place, the memory model in detail, and an honest list of
what the language does not do yet.

## Before you start

You'll need two things: a text editor and the Torvik toolchain.

Any editor works — Notepad, TextEdit, VS Code, Vim. Torvik source files end in `.tv` and are
plain text.

Installing the toolchain takes one command, covered at the top of
[Chapter 1](/guide-basics). Torvik runs on **Linux** and **Windows** (x86-64). macOS support
is planned once Apple hardware is available for credible testing.

Ready? [Start with Chapter 1 →](/guide-basics)
