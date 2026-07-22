---
title: Tour
description: A tour of the Torvik language - variables, functions, pattern matching, weave pipelines, error handling, and concurrency
---

# A tour of Torvik

Everything below is real, compiling Torvik. This is the quick tasting menu — for the full
story, start with the [Language Guide](/guide), which teaches the whole language from the
beginning.

## Variables

`set` declares a variable; `fixed` declares one that cannot be reassigned.

```torvik
set count: i64 = 3;
fixed name: str = "Odin";
count += 1;
```

## Functions

Functions are declared with `df`. Every parameter and return type is explicit.

```torvik
df square(n: i64) -> i64 {
    return n * n;
}
```

A parameter marked `^` is **optional** (callers may omit it, and it takes a
type-appropriate zero); a trailing `*` parameter is **variadic**, gathering the
remaining arguments into a `list`. Every argument is type-checked at the call site.

```torvik
df greet(name: str, ^greeting: str) -> void {
    check greeting == "" { greeting = "Hail"; }
    echo!("{greeting}, {name}!");
}

df loudest(*words: str) -> str {
    set best: str = "";
    each w in words { check len(w) > len(best) { best = w; } }
    return best;
}

greet("Astrid");            // Hail, Astrid!
greet("Bjorn", "Well met"); // Well met, Bjorn!
```

## Printing and interpolation

`echo!` prints with a newline and interpolates `{name}` directly - variables or full
expressions. `fmt` builds a string value the same way, and also takes positional `{}`
placeholders.

```torvik
echo!("Hello {name}!");
echo!("total: {price * qty}");
fixed line: str = fmt("{} + {} = {}", 2, 3, 5);
```

Interpolation belongs to `echo`, `echo!`, and `fmt` alone. A string literal anywhere
else is plain data - braces are just characters, so CSS, JSON, or template text needs
no escapes:

```torvik
fixed css: str = ".rune{color:red}";
fixed slot: str = "{{title}}";
```

## Conditionals and guards

`check` is Torvik's if; `fallback` is its else. `guard` states what must hold and
handles the exit path up front.

```torvik
check n > 9 {
    echo!("big");
} fallback {
    echo!("small");
}

guard len(name) > 0 fallback {
    echo!("a name is required");
    return;
}
```

## Aetts and pattern matching

An `aett` - named for the rune families - is an enumeration. `when` matches over it,
and the compiler checks every variant is covered.

```torvik
aett Status { Draft, Published, Archived }

df describe(s: Status) -> str {
    set out: str = "";
    when s {
        Status::Draft     => { out = "still being carved"; }
        Status::Published => { out = "raised for all to read"; }
        Status::Archived  => { out = "weathered but standing"; }
    }
    return out;
}
```

## Loops

`whilst` loops on a condition; `each` walks a range or a list.

```torvik
each i in 0..3 { echo!("run {i}"); }

fixed runes: list<str> = split("fehu,uruz,thurisaz", ",");
each r in runes { echo!(r); }
```

## The weave

The weave operator `~>` threads a value through a pipeline, left to right:

```torvik
fixed slug: str = " Hello, Torvik " ~> trim ~> lower ~> replace(" ", "-");
// hello,-torvik
```

## Errors as values

Fallible operations return a `result` - checked explicitly, no exceptions:

```torvik
set r: result<str> = try_readfile("saga.txt");
check is_err(r) {
    fixed why: str = err_msg(r);
    echo!("could not read: {why}");
    return;
}
fixed text: str = unwrap(r);
```

## Concurrency: ravens and bridges

`raven` spawns a function on its own OS thread; a `bridge` is a typed, buffered
channel between tasks. Arguments deep-copy at the spawn and values deep-copy on send,
so threads share no mutable state - and the compiler enforces that spawned functions
touch no globals.

```torvik
df fetch_len(path: str) -> i64 {
    return len(readfile(path));
}

df main() -> void {
    set h: task<i64> = raven fetch_len("saga.txt");
    // ... other work ...
    fixed n: i64 = join(h);
    echo!("{n} bytes");
}
```

## Collections

```torvik
set xs: list<i64> = list_new();
push(xs, 7);

set cfg: table<str, str> = table_new();
table_set(cfg, "title", "My Site");
fixed keys: list<str> = table_keys(cfg);   // sorted, so loops are reproducible
```

---

Ready to learn it properly? Start the [Language Guide](/guide) — it goes from no programming
experience at all through to concurrency and shipping real projects.

Or [install Torvik](/) and `rune new` your first project, and see how far the language goes
in [Vefna](/vefna), a real tool built with it.
