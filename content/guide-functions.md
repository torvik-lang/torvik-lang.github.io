---
title: Guide 3 - Functions
description: Define operations with df, pass parameters and return values, use optional and variadic parameters, write recursive functions, and pipe values with the weave operator.
---

# 3. Functions

*[← 2. Decisions and loops](/guide-flow) · [Guide index](/guide) · Next: [4. Collections and text →](/guide-collections)*

A **function** is a named piece of work. You've been writing one since your first program —
`main` — and calling others like `echo!` and `len`. Now you'll write your own.

Functions are how programs stay manageable. They let you name an idea once, use it
everywhere, and change it in one place.

## Defining a function

```torvik
df square(n: i64) -> i64 {
    return n * n;
}

df main() -> void {
    echo!(square(5));     // 25
    echo!(square(12));    // 144
}
```

The shape is:

```
df NAME(PARAMETERS) -> RETURN_TYPE { BODY }
```

- **`df`** starts the definition.
- **`square`** is the name — pick a verb or a noun that says what it does.
- **`(n: i64)`** is the **parameter list**: named inputs, each with a type.
- **`-> i64`** is the **return type**: the kind of value it hands back.
- **`return`** sends a value back and ends the function immediately.

A function that returns nothing uses `-> void` and simply has no `return` value:

```torvik
df greet(name: str) -> void {
    echo!("Hail, {name}!");
}
```

Multiple parameters are separated by commas:

```torvik
df add(a: i64, b: i64) -> i64 {
    return a + b;
}

df describe(name: str, age: i64, active: bool) -> str {
    return fmt("{name} ({age}) active={active}");
}
```

## Calling functions

Write the name and pass values in parentheses:

```torvik
fixed total: i64 = add(3, 4);
greet("Freya");
echo!(add(square(3), 1));      // calls nest freely: 10
```

Arguments are matched **by position**: the first value goes to the first parameter. Torvik
checks both the count and the types at compile time, so passing a string where a number is
expected — or three arguments to a two-parameter function — is a clean error, never a
run-time surprise.

Functions may call functions defined **later** in the file, and they may call themselves.
Order doesn't matter:

```torvik
df main() -> void {
    echo!(double(21));       // fine - double is defined below
}

df double(n: i64) -> i64 {
    return n * 2;
}
```

## Why functions matter

Consider code that computes a total three times in three places. When the shipping rule
changes, you have three edits to find and one to forget. With a function there's one place:

```torvik
df order_total(price: i64, qty: i64) -> i64 {
    set total: i64 = price * qty;
    check total < 500 { total += 50; }   // shipping
    return total;
}
```

A good function does **one thing** and its name says what. If you find yourself writing "and"
in the name — `validate_and_save` — that's usually two functions.

## Returning early

`return` exits immediately. Combined with `check` or `guard`, this keeps functions flat:

```torvik
df classify(n: i64) -> str {
    check n < 0  { return "negative"; }
    check n == 0 { return "zero"; }
    return "positive";
}
```

Every path must return a value (unless the function is `-> void`). The compiler enforces
this — a path that falls off the end without returning is a compile error, not a garbage
value.

## Optional parameters: `^`

Sometimes a parameter has a sensible default. Mark it with `^` and callers may leave it off:

```torvik
df greet(name: str, ^greeting: str) -> void {
    set g: str = greeting;
    check g == "" { g = "Hail"; }
    echo!("{g}, {name}!");
}

df main() -> void {
    greet("Astrid");               // Hail, Astrid!
    greet("Bjorn", "Well met");    // Well met, Bjorn!
}
```

An omitted optional gets a **type-appropriate zero**: `0` for integers, `0.0` for `f64`,
`false` for `bool`, and `""` for `str`. Check for that value to apply your own default, as
above.

Two rules:

- **Required parameters come first.** Optionals must be at the end, so a positional call is
  never ambiguous. Putting a required parameter after an optional one is a compile error.
- Optionals are scalar or `str` types.

Passing too few or too many arguments gives you a clean range error:

```
error: function 'greet' expects 1 to 2 argument(s) but got 3.
```

## Variadic parameters: `*`

When a function should accept *any number* of trailing values, mark the **last** parameter
with `*`. Those arguments arrive as a list you can iterate:

```torvik
df join_words(sep: str, *words: str) -> str {
    set out: str = "";
    set first: bool = true;

    each w in words {
        check first {
            out = w;
            first = false;
        } fallback {
            out = str_concat(out, sep, w);
        }
    }
    return out;
}

df main() -> void {
    echo!(join_words(", "));                       // (empty)
    echo!(join_words(", ", "one"));                // one
    echo!(join_words(", ", "one", "two", "three"));// one, two, three
}
```

A variadic parameter must be **last**, and there can be only one. Its element type is `str`
or an integer type.

All three kinds combine, in this order — required, then optional, then variadic:

```torvik
df log_line(level: str, ^tag: str, *parts: str) -> void {
    set msg: str = str_concat("[", level, "]");
    check tag != "" { msg = str_concat(msg, "(", tag, ")"); }
    each p in parts { msg = str_concat(msg, " ", p); }
    echo!(msg);
}

log_line("INFO");                                 // [INFO]
log_line("WARN", "net");                          // [WARN](net)
log_line("ERR", "disk", "read", "failed");        // [ERR](disk) read failed
```

## Arguments are type-checked

Every argument is checked against its parameter at the call site. A definite mismatch — a
number where a string is expected, a decimal where an integer is expected, a list where a
scalar is expected — is a clean, located compile error:

```
error: argument 1 of 'greet' has the wrong type - this parameter expects str, but got a number.
```

This matters more than it sounds. In a language without that check, passing the wrong type
produces a garbage value that flows onward and fails somewhere else entirely, hours later.
Here it fails at the line where you made the mistake.

## Scope

A variable declared inside a function exists only inside that function. Blocks nest inward:
an inner block can see outer variables, but not the reverse.

```torvik
df demo() -> void {
    fixed outer: i64 = 1;

    check true {
        fixed inner: i64 = 2;
        echo!(outer);         // fine - outer is visible here
    }

    // echo!(inner);          // error: inner no longer exists
}
```

Parameters are variables scoped to their function. Two functions can each have a parameter
called `n` with no relationship between them — a function's inputs and locals are private to
it. That isolation is exactly what makes functions safe to reuse.

Variables declared at the **top level**, outside any function, are **globals**, visible
everywhere. Use them sparingly: a value anything can change is a value you can't reason
about. (They also can't be touched by concurrent tasks — see
[Chapter 6](/guide-concurrency).)

## Recursion

A function may call **itself**. This is the natural way to express problems that contain
smaller copies of themselves.

```torvik
df factorial(n: i64) -> i64 {
    check n <= 1 { return 1; }         // base case
    return n * factorial(n - 1);       // recursive case
}

echo!(factorial(5));   // 120
```

Every recursive function needs two parts:

1. A **base case** that returns without recursing. This is what stops it.
2. A **recursive case** that moves *toward* the base case.

Get the base case wrong and the function calls itself forever until the program runs out of
stack space and crashes. When a recursive function misbehaves, check the base case first.

Tracing `factorial(4)` by hand:

```
factorial(4) = 4 * factorial(3)
             = 4 * (3 * factorial(2))
             = 4 * (3 * (2 * factorial(1)))
             = 4 * (3 * (2 * 1))
             = 24
```

Recursion shines on naturally nested structures — directory trees, nested data, divide-and-
conquer algorithms. For plain counting, a loop is simpler and faster. Use whichever states
the problem more honestly.

## The weave operator: `~>`

When you apply several transformations in sequence, nested calls read inside-out:

```torvik
echo!(upper(replace(trim(raw), ",", "-")));    // read from the middle outward
```

The **weave** operator pipes a value left to right instead. `x ~> f` is `f(x)`:

```torvik
fixed slug: str = raw ~> trim ~> replace(",", "-") ~> upper;
```

Same result, read in the order it happens: take `raw`, trim it, replace commas, uppercase it.

A stage with arguments inserts the woven value as the **first** argument, so
`x ~> f(a, b)` means `f(x, a, b)`:

```torvik
df addn(x: i64, n: i64) -> i64 { return x + n; }

set v: i64 = 5;
echo!(v ~> addn(3) ~> addn(10));    // 18
```

One-argument builtins like `trim` and `upper` are written bare, with no parentheses. Weave
results are typed and checked: weaving an `i64` into a function whose first parameter is
`str` is a compile error, and argument counts are verified including the inserted value.

## Putting it together

```torvik
df is_even(n: i64) -> bool {
    return n % 2 == 0;
}

df sum_range(from: i64, to: i64, ^evens_only: bool) -> i64 {
    set total: i64 = 0;
    each n in from..+to {
        check evens_only {
            check is_even(n) { total += n; }
        } fallback {
            total += n;
        }
    }
    return total;
}

df report(label: str, *values: i64) -> void {
    set total: i64 = 0;
    set count: i64 = 0;
    each v in values {
        total += v;
        count += 1;
    }
    check count == 0 {
        echo!("{label}: no values");
        return;
    }
    echo!("{label}: {count} values, total {total}, mean {total / count}");
}

df main() -> void {
    echo!(sum_range(1, 10));            // 55
    echo!(sum_range(1, 10, true));      // 30 - evens only

    report("empty");
    report("scores", 90, 75, 88);
}
```

## Things to try

1. Write `min(a, b)` and `max(a, b)`, then use them together.
2. Write `celsius_to_f(c: f64) -> f64` and print a small conversion table with `each`.
3. Give a `greet` function an optional `^title` parameter that prefixes the name when
   supplied.
4. Write a variadic `longest(*words: str) -> str` returning the longest word.
5. Write `fib(n)` recursively. Then write it with a loop. Time both mentally for `fib(40)` —
   why is one dramatically slower?
6. Take a nested expression three calls deep and rewrite it with `~>`. Decide which you find
   clearer.

---

*[← 2. Decisions and loops](/guide-flow) · [Guide index](/guide) · Next: [4. Collections and text →](/guide-collections)*
