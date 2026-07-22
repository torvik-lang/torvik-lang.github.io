---
title: Guide 6 - Concurrency
description: Run work on several threads with raven tasks and typed bridge channels - a model where data races are not something you avoid, but something you cannot write.
---

# 6. Concurrency: ravens and bridges

*[← 5. Errors and safety](/guide-errors) · [Guide index](/guide) · Next: [7. Modules and projects →](/guide-projects)*

Modern processors have many cores. A program that runs on one is leaving most of the machine
idle. **Concurrency** is doing several things at once.

It's also the area where most languages hurt you. Two threads touching the same data at the
same time produce a **data race**: a bug that appears one run in a thousand, vanishes when
you add a print statement, and cannot be reproduced on demand.

Torvik's answer is structural. **Tasks share no mutable state.** Values are copied as they
cross a thread boundary, so a data race isn't something you avoid by being careful — it's
something the language gives you no way to express.

Two ideas do the work: a **raven** carries a task to its own thread and comes back with what
it found; a **bridge** is a typed channel carrying values between tasks.

## Ravens: spawning tasks

`raven` prefixes a call to one of your `df` functions and runs it on a new OS thread.

**Fire and forget** — start it and move on:

```torvik
raven log_event("started");     // runs on its own thread; main does not wait
```

**With a handle** — capture it so you can collect the result later. The handle's type is
`task<T>`, where `T` is the function's return type. `join` waits for it to finish and hands
back the value:

```torvik
df slow_sum(from: i64, to: i64) -> i64 {
    set total: i64 = 0;
    each n in from..+to { total += n; }
    return total;
}

df main() -> void {
    set h: task<i64> = raven slow_sum(1, 1000000);

    echo!("working on something else meanwhile...");

    fixed result: i64 = join(h);      // wait, take the value
    echo!("sum: {result}");
}
```

Between the `raven` line and the `join` line, both things run at once. That gap is where the
speedup lives — do as much other work there as you can.

Run several at once and join them all:

```torvik
df main() -> void {
    set a: task<i64> = raven slow_sum(1, 500000);
    set b: task<i64> = raven slow_sum(500001, 1000000);

    fixed total: i64 = join(a) + join(b);
    echo!("total: {total}");
}
```

Both tasks run in parallel; the joins collect them in turn.

A few rules:

- `join(h)` is an ordinary expression, so it composes: `check join(h) == "ok" { ... }`.
- A `task<void>` is joined as a **statement**: `join(h);` waits and yields nothing.
- Every handle may be joined **exactly once**. A second join is a clean panic — the result
  has already been taken.
- A bare `join(h);` on a value-returning task is fine: it waits and discards.

Any crossable type can be a task result: every integer width, `f64`, `bool`, `str`,
`i128`/`u128`, and `aett` values.

## Arguments are copied at the spawn

When you write `raven greet(name)`, the argument is **deep-copied at the moment the spawn
runs**, on your thread. After that line, the task owns its copy and you own yours:

```torvik
set name: str = "original";
set h: task<str> = raven greet(name);

name = "changed";               // the task already has its own copy
echo!(join(h));                 // greeted "original", not "changed"
```

This is the whole reason the model needs no locks on ordinary values: only one thread can
ever see a given string or box. It also removes a classic bug — the loop that spawns tasks,
mutates a shared variable, and has every task see the last value.

Collections (`list`, `table`, `bag`) and `result` values do **not** cross into a task in this
version. Pass the scalars and strings a task needs and rebuild structure inside it. The one
deliberate exception is a bridge, below, which is shared on purpose.

## The self-contained rule

A raven carries everything it needs in its claws. **A function that is ever spawned — and
every function it transitively calls — may not read or write a global variable.** The
compiler checks this and names the spawn, the function, and the global:

```torvik
set counter: i64 = 0;

df tick() -> i64 { return counter; }    // reads a global

df main() -> void {
    set h: task<i64> = raven tick();    // error: 'tick' is not self-contained
    echo!(join(h));
}
```

The reason is precise, and worth understanding. Even *reading* a global string from two
threads races on its **reference count**, which isn't atomic. Torvik could make every
refcount in the language atomic — and every single-threaded program would pay for it. Instead
it requires tasks to be self-contained and keeps the runtime lock-free.

The fix is always the same: pass what the task needs as an argument.

```torvik
df tick(start: i64) -> i64 { return start + 1; }

df main() -> void {
    fixed counter: i64 = 0;
    set h: task<i64> = raven tick(counter);   // fine - it's an argument now
    echo!(join(h));
}
```

This rule can only loosen in future versions, never tighten — code that compiles today keeps
compiling.

## Panics in tasks

If a task panics — an out-of-bounds index, a failed `vow`, a `halt` — the **whole process
stops immediately**, with the usual message, exactly as on the main thread. A background
failure is never silently swallowed or deferred until you happen to join.

For failure you want to *handle*, return a `result<T>` from the task and inspect it after
joining, like anywhere else:

```torvik
df fetch(id: i64) -> result<str> {
    check id < 0 { return err("bad id"); }
    return ok(fmt("record-{id}"));
}

df main() -> void {
    set h: task<result<str>> = raven fetch(-1);
    fixed r: result<str> = join(h);

    check is_err(r) {
        echo!("task failed: {err_msg(r)}");
    } fallback {
        echo!(unwrap(r));
    }
}
```

## Bridges: channels between tasks

Joining collects one value at the end. A **bridge** carries a stream of values while tasks
are still running. It's a typed, buffered queue that several threads can use at once — the
one object tasks share, with its own lock protecting its interior.

```torvik
set ch: bridge<str> = bridge_new(8);    // capacity 8 (must be >= 1)

send(ch, "hello");                       // copies the value in; blocks if full
fixed msg: str = recv(ch);               // takes one out; blocks if empty
```

Values **deep-copy on send**, the same guarantee as spawn arguments: once a value crosses,
sender and receiver own independent copies.

The blocking behaviour is a feature. If the buffer is full, `send` waits — which throttles a
fast producer to the speed of its consumer instead of growing memory without bound.

A bridge passed **into** a task is the one argument that isn't copied: the task receives the
bridge itself (a private copy of a channel would be pointless). That's what makes it the
shared object, and why it carries its own lock.

## Closing a bridge and the worker loop

`bridge_close(ch)` says "no more values will be sent." It's idempotent and wakes anyone
waiting.

After a bridge is closed *and* emptied, `recv` panics — reading past the end of a finished
stream is a mistake. To drain a stream to its natural end, use `try_recv`, which returns a
`result<T>`: ok while data remains, and a single err once the bridge is closed and drained.
That err is your stop signal:

```torvik
df produce(ch: bridge<i64>, n: i64) -> void {
    set i: i64 = 0;
    whilst i < n {
        send(ch, i);
        i += 1;
    }
    bridge_close(ch);                    // done producing
}

df main() -> void {
    set ch: bridge<i64> = bridge_new(4);

    raven produce(ch, 10);               // producer on its own thread

    set total: i64 = 0;
    whilst true {
        set r: result<i64> = try_recv(ch);
        check is_err(r) { break; }       // closed and drained: stop cleanly
        total += unwrap(r);
    }

    echo!("sum: {total}");               // 45
}
```

That's the canonical worker loop, and it's worth memorising: **`try_recv` in a loop, break on
err.**

`send` into a closed bridge panics — closing means you promised not to. Multiple producers
and multiple consumers on one bridge are fully supported: the bridge's lock serialises them,
and each value is received **exactly once by exactly one consumer**.

## What crosses, and what waits

- **Across a spawn or bridge:** every integer width, `u64`, `f64`, `bool`, `str`,
  `i128`/`u128`, and `aett` values. Collections and `result` don't cross yet.
- **`bridge_new(cap)`** needs `cap >= 1`; unbuffered rendezvous channels are a later
  addition, and a smaller capacity panics.
- **Ordering** is whatever `join` and bridge blocking impose. There is no hidden ordering —
  and no sleeps are ever needed to make concurrent Torvik deterministic. Force the ordering
  you want by joining, or by the sequence of sends and receives.

If you ever find yourself reaching for a sleep to "let a task finish," you want a `join` or a
`recv` instead.

## Putting it together

A pipeline: several workers process items and report through one bridge while the main thread
collects.

```torvik
df worker(ch: bridge<str>, id: i64, count: i64) -> void {
    set i: i64 = 0;
    whilst i < count {
        send(ch, fmt("worker {id} finished item {i}"));
        i += 1;
    }
}

df main() -> void {
    set ch: bridge<str> = bridge_new(16);

    raven worker(ch, 1, 3);
    raven worker(ch, 2, 3);

    set received: i64 = 0;
    whilst received < 6 {                 // two workers, three items each
        fixed msg: str = recv(ch);
        echo!(msg);
        received += 1;
    }

    bridge_close(ch);
    echo!("all {received} items reported");
}
```

Both workers run in parallel, their messages interleave in whatever order they finish, and
every message arrives exactly once. No locks, no shared counters, no races — there was never
any shared mutable state to protect.

## Things to try

1. Spawn two tasks computing sums over different ranges and join both.
2. Spawn a task, then try to `join` it twice. Read the panic.
3. Write a function that reads a global and try to spawn it. Read the error, then fix it by
   passing the value as an argument.
4. Build a producer/consumer with `try_recv` and confirm it stops cleanly on close.
5. Make a bridge with capacity 1 and a producer much faster than the consumer. Convince
   yourself the producer is being throttled.
6. Spawn a task that calls `vow` with a false condition and watch the whole process stop.
7. Run three workers into one bridge and count the total messages. Note that the *order*
   varies between runs but the *count* never does.

---

*[← 5. Errors and safety](/guide-errors) · [Guide index](/guide) · Next: [7. Modules and projects →](/guide-projects)*
