---
title: Guide 2 - Decisions and loops
description: Branch with check and fallback, use the ternary, exit early with guard, repeat work with whilst and each, and match patterns with aett and when.
---

# 2. Making decisions and repeating work

*[← 1. First steps](/guide-basics) · [Guide index](/guide) · Next: [3. Functions →](/guide-functions)*

So far every program has run straight down the page. Real programs need to choose between
paths and repeat work. That's this chapter.

## Conditions: `check` and `fallback`

`check` runs a block only when a condition is true. It's what other languages call `if`.

```torvik
df main() -> void {
    fixed temperature: i64 = 30;

    check temperature > 25 {
        echo!("It's warm.");
    }
}
```

The condition sits between `check` and the `{`. No parentheses needed (though you can add
them for clarity). The block runs only if the condition is `true`.

Add `fallback` for the "otherwise" path:

```torvik
check temperature > 25 {
    echo!("It's warm.");
} fallback {
    echo!("It's cool.");
}
```

Exactly one of those two blocks runs. Chain them for several cases:

```torvik
check score >= 90 {
    echo!("excellent");
} fallback {
    check score >= 70 {
        echo!("good");
    } fallback {
        echo!("keep practising");
    }
}
```

Conditions combine with `&&`, `||`, and `!`:

```torvik
check age >= 13 && age < 20 {
    echo!("teenager");
}

check is_admin || is_owner {
    echo!("has access");
}

check !is_locked {
    echo!("you may enter");
}
```

Since v1.4.0 a boolean expression can also be **stored**, which often reads better than
burying the logic in the condition:

```torvik
fixed can_edit: bool = is_admin || is_owner;
fixed ready: bool = is_loaded && !is_locked && has_data;

check can_edit {
    echo!("editing allowed");
}
```

A boolean-returning call can be compared against a literal directly:

```torvik
check contains(line, "error") == false {
    echo!("line is clean");
}
```

## The ternary: `?>` and `!>`

Sometimes you want to *choose a value*, not run a block. Torvik's conditional expression is
`condition ?> value_if_true !> value_if_false`. Read `?>` as "then" and `!>` as "else."

```torvik
set n: i64 = 7;
fixed parity: str = n % 2 == 0 ?> "even" !> "odd";   // "odd"

echo!(n > 0 ?> "positive" !> "non-positive");
```

Compare that with the `check` version — four lines and a mutable variable versus one line and
a `fixed` one. When you're picking between two values, the ternary is usually clearer.

Ternaries chain to the right for several cases:

```torvik
fixed sign: str = n > 0 ?> "positive"
               !> n < 0 ?> "negative"
               !> "zero";
```

Both branches must be the same kind of type — two numbers, or two strings, not one of each.

The ternary **short-circuits**: only the branch actually taken is evaluated. That makes it
safe as a guard:

```torvik
fixed safe: i64 = d != 0 ?> 100 / d !> 0;   // never divides by zero
```

Don't nest ternaries more than about two deep. Past that, `check` blocks or a `when` are
kinder to read.

## Early exits: `guard`

`guard` states a condition that **must** hold for the code to continue. If it's false, the
`fallback` block runs — usually to return early.

```torvik
df reciprocal_label(n: i64) -> str {
    guard n != 0 fallback {
        return "undefined";
    }
    // Past this point, n is guaranteed non-zero.
    return "defined";
}
```

The value of `guard` is that it keeps the **happy path unindented**. Compare:

```torvik
// With guards - the real work stays at the left margin
df process(name: str, age: i64) -> str {
    guard len(name) > 0 fallback { return "name required"; }
    guard age >= 0      fallback { return "age cannot be negative"; }
    guard age < 150     fallback { return "age looks wrong"; }

    return fmt("{name} is {age}");
}
```

Every precondition is stated once, up front, next to its error message. The alternative —
nesting three `check` blocks — pushes the actual work three levels deep and puts each error
message far from the test that triggers it.

If a guard's condition is compound, wrap it in parentheses:

```torvik
guard (a > 0 && b > 0) fallback { return "both must be positive"; }
```

## Loops: `whilst`

`whilst` repeats a block as long as a condition stays true.

```torvik
set i: i64 = 0;
whilst i < 5 {
    echo!(i);       // 0 1 2 3 4
    i += 1;
}
```

Three things have to be right, and getting any of them wrong is the classic beginner bug:

1. **Initialize** — `i` starts at `0`.
2. **Test** — the loop continues while `i < 5`.
3. **Advance** — `i += 1` moves toward the end.

Forget step 3 and the condition never becomes false: the loop runs forever. If a program
hangs, a missing advance is the first thing to check. (Ctrl-C stops it.)

`whilst true { ... }` is a deliberate infinite loop, which is fine when something inside
breaks out of it.

## Loops: `each`

When you want to count through a range of numbers, `each` says it more directly.

```torvik
each i in 0..5 {
    echo!(i);       // 0 1 2 3 4
}
```

`START..END` is **exclusive** — the end value is not included. `0..5` gives you five
iterations: 0, 1, 2, 3, 4. That pairs perfectly with zero-based indexing.

For an inclusive range, use `..+`:

```torvik
each i in 0..+5 {
    echo!(i);       // 0 1 2 3 4 5
}
```

`each` handles the counter for you — there's no variable to forget to advance, so an `each`
loop can't spin forever. Prefer it over `whilst` whenever you know the range in advance.

```torvik
set sum: i64 = 0;
each k in 1..+100 {
    sum += k;
}
echo!("1 to 100 sums to {sum}");   // 5050
```

You can also iterate a **list** directly, which we'll use constantly in
[Chapter 4](/guide-collections):

```torvik
each word in words {
    echo!(word);
}
```

## `break` and `continue`

`break` leaves the loop immediately. `continue` skips the rest of this pass and starts the
next one.

```torvik
each i in 0..10 {
    check i == 3 { break; }    // stops entirely at 3
    echo!(i);                  // 0 1 2
}

each i in 0..5 {
    check i == 2 { continue; } // skips just 2
    echo!(i);                  // 0 1 3 4
}
```

Both work in `whilst` and `each`. In an `each` loop, `continue` still advances the counter,
so it can't cause an infinite loop.

A common shape — search a range and stop at the first hit:

```torvik
set found: i64 = -1;
each i in 0..len(xs) {
    check xs[i] == target {
        found = i;
        break;                 // no point looking further
    }
}
```

## Named values: `aett`

An **aett** is a family of named values — an enumeration. It's named for the *ættir*, the
families the runes of the futhark are grouped into.

Use one whenever a value has a small, fixed set of possibilities. Instead of remembering that
status `0` means pending and `2` means closed, you name them:

```torvik
aett Status { Pending, Active, Closed }

df main() -> void {
    set s: Status = Status::Pending;
    echo!(s);              // 0 - variants are i64 ordinals in declaration order
    echo!(typeof(s));      // Status

    s = Status::Closed;
    check s == Status::Closed {
        echo!("closed");
    }
}
```

Declare an aett at the **top level** of the file, not inside a function. Access variants with
`::`.

Aett values are backed by `i64` ordinals, so they print, compare, store in lists, and pass to
functions like any integer — while the aett name works as a real type annotation for
variables, parameters, and return types.

## Pattern matching: `when`

`when` matches a value against a set of patterns. Arms use `=>`, with either a single
statement or a block on the right:

```torvik
aett Status { Pending, Active, Closed }

df describe(s: Status) -> str {
    when s {
        Status::Pending => return "waiting";
        Status::Active  => return "live";
        Status::Closed  => return "done";
    }
    return "";
}
```

Here's the part that makes `when` genuinely valuable. When the value being matched is
aett-typed and there's no `fallback` arm, the compiler checks **exhaustiveness**: if you
haven't covered every variant, it's a clean compile error naming exactly which one you
missed.

That turns a whole category of bug into a compile error. Add a fourth variant to `Status` six
months from now, and the compiler immediately points at every `when` that needs updating —
instead of your program silently taking a wrong branch in production.

`when` also matches integers, using literal patterns. Here a `fallback` arm is **required**,
because the compiler can't prove that a handful of literals cover every possible integer:

```torvik
when n {
    1        => echo!("one");
    7        => echo!("seven");
    fallback => echo!("many");
}
```

Arms can be blocks when you need more than one statement:

```torvik
when s {
    Status::Active => {
        echo!("running");
        start_timer();
    }
    fallback => echo!("not running");
}
```

As everywhere in Torvik, `fallback` comes last.

## Putting it together

A number-guessing scorer using most of this chapter:

```torvik
aett Verdict { TooLow, Correct, TooHigh }

df judge(guess: i64, secret: i64) -> Verdict {
    check guess < secret { return Verdict::TooLow; }
    check guess > secret { return Verdict::TooHigh; }
    return Verdict::Correct;
}

df main() -> void {
    fixed secret: i64 = 42;
    set guesses: i64 = 0;
    set current: i64 = 0;

    each attempt in 1..+10 {
        current = attempt * 9;          // stand-in for real input
        guesses += 1;

        fixed v: Verdict = judge(current, secret);

        when v {
            Verdict::Correct => {
                echo!("Got it in {guesses}: {current}");
                break;
            }
            Verdict::TooLow  => echo!("{current} is too low");
            Verdict::TooHigh => echo!("{current} is too high");
        }
    }
}
```

Note the `when` has no `fallback` and covers all three variants — the compiler verified that
for you.

## Things to try

1. Print the numbers 1 to 20, but for multiples of 3 print `"fizz"` instead. (`%` is your
   friend.)
2. Rewrite that using a ternary inside the `echo!`.
3. Write a function that grades a score 0–100 into an `aett` of `Fail`, `Pass`, `Merit`,
   `Distinction`, then `when` over it. Delete one arm and read the error.
4. Sum the even numbers between 1 and 100 using `each` and `continue`.
5. Take the `process` guard example and rewrite it with nested `check` blocks instead. Decide
   for yourself which you'd rather maintain.
6. Write a `whilst` loop that deliberately never ends, run it, and stop it with Ctrl-C. Then
   fix it. Knowing what a hang feels like is useful.

---

*[← 1. First steps](/guide-basics) · [Guide index](/guide) · Next: [3. Functions →](/guide-functions)*
