---
title: Guide 5 - Errors and safety
description: Handle failure explicitly with result, ok, and err; assert invariants with vow and halt; read compile warnings; and use the narrow unsafe prefix.
---

# 5. Errors, safety, and the compiler on your side

*[← 4. Collections and text](/guide-collections) · [Guide index](/guide) · Next: [6. Concurrency →](/guide-concurrency)*

Things fail. Files are missing, input is malformed, a number won't parse. How a language
handles failure shapes how trustworthy programs written in it are.

Torvik has no exceptions. Nothing is thrown past you invisibly. A function that can fail says
so **in its return type**, and you deal with it at the call site.

## `result<T>`

A `result<T>` is either an **ok** carrying a value of type `T`, or an **err** carrying a
message (and optionally a numeric code).

```torvik
df safe_div(a: i64, b: i64) -> result<i64> {
    check b == 0 { return err("division by zero"); }
    return ok(a / b);
}
```

The signature is the documentation: `-> result<i64>` says "an integer, or an explanation of
why not." A caller can't accidentally ignore that.

### Consuming a result

```torvik
df main() -> void {
    set r: result<i64> = safe_div(10, 2);

    check is_ok(r) {
        echo!("answer: {unwrap(r)}");
    } fallback {
        echo!("failed: {err_msg(r)}");
    }
}
```

The full set:

| Function | Purpose |
|---|---|
| `is_ok(r)` / `is_err(r)` | Test which state it's in |
| `unwrap(r)` | Take the value — **halts** if it's an err |
| `unwrap_or(r, default)` | Take the value, or a fallback if it's an err |
| `err_msg(r)` | The error message |
| `err_code(r)` | The numeric error code, if one was set |

`unwrap` halts the program when the result is an err, so only use it after checking, or when
an err genuinely means the program cannot continue. When you have a sensible default,
`unwrap_or` says so in one line:

```torvik
fixed port: i64 = unwrap_or(try_toint(text), 8080);
```

You can attach a code alongside the message with `err(code, message)`, which is useful when a
caller needs to branch on the kind of failure rather than its wording.

`result<i64>`, `result<str>`, and `result<f64>` are supported as parameters, return types,
and locals.

### Builtins that return results

Several operations come in a result-returning form — the same work, but failure is a value
you inspect instead of a halt:

```torvik
fixed content: result<str> = try_readfile("config.txt");
fixed n: result<i64> = try_toint(user_input);
fixed f: result<f64> = try_tofloat(user_input);
```

Use these whenever failure is *expected*: user input, files that may not exist, network
data. Use the plain versions (`toint`, `readfile`) only when failure means your assumptions
are already broken.

```torvik
df read_port(text: str) -> i64 {
    fixed parsed: result<i64> = try_toint(text);

    check is_err(parsed) {
        echo!("'{text}' is not a number, using 8080");
        return 8080;
    }

    fixed port: i64 = unwrap(parsed);

    check port < 1 {
        echo!("port must be positive, using 8080");
        return 8080;
    }
    check port > 65535 {
        echo!("port too large, using 8080");
        return 8080;
    }

    return port;
}
```

### Passing failure upward

A function that can't handle a failure itself returns it to whoever can:

```torvik
df load_config(path: str) -> result<str> {
    fixed raw: result<str> = try_readfile(path);
    check is_err(raw) {
        return err(fmt("could not read {path}: {err_msg(raw)}"));
    }

    fixed text: str = unwrap(raw);
    check len(trim(text)) == 0 {
        return err(fmt("{path} is empty"));
    }

    return ok(text);
}
```

Each layer adds context. By the time it reaches `main`, the message says what was being
attempted, not just what went wrong at the bottom.

## `vow` and `halt`

`result` is for failures you **expect**. Some conditions instead mean your program's
assumptions are broken — continuing would produce nonsense.

`halt(message)` prints and exits immediately with a non-zero status:

```torvik
df main() -> void {
    halt("unrecoverable: configuration missing");
    // nothing after this runs
}
```

`vow(condition, message)` asserts an invariant: if the condition is false it prints and
exits; if true, execution continues.

```torvik
set balance: i64 = compute_balance();
vow(balance >= 0, "balance must never be negative");
echo!("invariant held");
```

Use `vow` to state things that should be **impossible**, not things that might reasonably
happen. Bad user input is a `result`. A negative balance after a series of checked
transactions is a `vow` — if it fails, there's a bug, and stopping immediately with a clear
message beats silently corrupting data.

A useful rule of thumb:

| Situation | Tool |
|---|---|
| Might reasonably happen (bad input, missing file) | `result<T>` |
| Should be impossible if the code is correct | `vow` |
| Cannot continue, no assertion needed | `halt` |

## Compile warnings

Warnings flag code that is legal but probably not what you meant. They never fail the
build — the binary is produced either way — and each comes with the same line-and-caret
display as an error.

**Unused variable** — a binding that's never read:

```torvik
fixed total: i64 = compute();      // warning if never used
fixed _total: i64 = compute();     // underscore prefix: deliberate, no warning
```

Usually this means a typo or leftover code. Occasionally you want the call's side effect and
not its value — the underscore says so.

**Unreachable code** — a statement after `return`, `break`, `continue`, `halt`, or `exit` in
the same block. Almost always a misplaced `return`.

**Unused result** — a bare statement call of a non-void function:

```torvik
init(1);                       // warning: the i64 result is unused
fixed r: i64 = init(1);        // bind it
fixed _r: i64 = init(1);       // or say the discard is deliberate
```

This one catches a genuinely sneaky bug: calling something that *returns* a status and
ignoring it.

**Deprecations** — builtins scheduled for removal warn at every call site with the
replacement to use.

Warnings are shown even in quiet builds (`-q`), because a diagnostic you don't see is
useless. `torvc --no-warn` turns them off.

### Controlling warnings in a file

At the top of a file:

```torvik
!@NO_WARN;                    // suppress all warnings for this compilation
!@ALLOW[unused_variable];     // suppress one category (stackable, one per line)
```

Categories: `unused_variable`, `unreachable_code`, `deprecated`, `unused_result`. A typo or
unknown category is a **clean compile error**, not a silent no-op — a directive that
quietly did nothing would be exactly the bug class Torvik exists to eliminate.

Treat warnings as errors you haven't fixed yet. A codebase with warnings scrolling past is a
codebase where the real one gets missed.

## The `unsafe` prefix

`unsafe` in Torvik is **not** a general "anything goes" region. It's a single-statement
prefix that opts into one specific operation the compiler would otherwise reject.

Today that operation is **wrapping an out-of-range integer literal into a sized type**:

```torvik
set b: i8 = 200;          // error: 200 is out of range for i8 (valid -128 to 127)
unsafe set b: i8 = 200;   // allowed - you take responsibility; b wraps to -56
unsafe x = 300;           // also works on an assignment
```

By default the compiler refuses, because silently truncating changes the value and hides
bugs. When wrapping is genuinely what you want — packing bytes, matching a wire format — you
say so explicitly, and the `unsafe` keyword marks it for every future reader.

It prefixes exactly one declaration or assignment. Using it anywhere else is a clean compile
error rather than a no-op:

```torvik
unsafe echo!("hi");   // error: unsafe can only prefix a declaration or assignment
```

Future releases may extend `unsafe` to other rejected-by-default operations. Each will be an
explicit, documented opt-in like this one — never a blanket escape hatch.

## Putting it together

```torvik
df parse_setting(line: str) -> result<str> {
    fixed clean: str = trim(line);

    check len(clean) == 0 { return err("empty line"); }
    check starts(clean, "#") { return err("comment line"); }

    fixed eq: i64 = find(clean, "=");
    check eq < 0 { return err(fmt("no '=' in: {clean}")); }

    fixed key: str = trim(substr(clean, 0, eq));
    check len(key) == 0 { return err(fmt("empty key in: {clean}")); }

    return ok(key);
}

df main() -> void {
    set lines: list<str> = list_new();
    push(lines, "port = 8080");
    push(lines, "# a comment");
    push(lines, "");
    push(lines, "broken line");

    set good: i64 = 0;
    set bad: i64 = 0;

    each line in lines {
        fixed r: result<str> = parse_setting(line);

        check is_ok(r) {
            good += 1;
            echo!("key: {unwrap(r)}");
        } fallback {
            bad += 1;
            echo!("skipped ({err_msg(r)})");
        }
    }

    vow(good + bad == len(lines), "every line must be accounted for");
    echo!("");
    echo!("{good} parsed, {bad} skipped");
}
```

Every failure is visible, named, and handled — and the `vow` states an invariant that should
never be false.

## Things to try

1. Write `safe_sqrt(x: f64) -> result<f64>` returning an err for negative input.
2. Use `try_toint` to parse a list of strings, counting successes and failures.
3. Write a function returning `err(code, message)`, and have the caller branch on
   `err_code`.
4. Deliberately trigger each warning — unused variable, unreachable code, unused result —
   and read what the compiler says.
5. Add `!@ALLOW[unused_variable];` to that file and watch one disappear. Then typo the
   category and read the error.
6. Try `set b: i8 = 200;`, read the error, then add `unsafe` and print the result. Work out
   *why* it becomes -56.
7. Take a function using `unwrap` directly and rewrite it with `unwrap_or`.

---

*[← 4. Collections and text](/guide-collections) · [Guide index](/guide) · Next: [6. Concurrency →](/guide-concurrency)*
