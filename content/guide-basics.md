---
title: Guide 1 - First steps
description: Install Torvik, write your first program, and learn the raw material of all code - comments, variables, types, operators, and printing with interpolation.
---

# 1. First steps

*[← Guide index](/guide) · Next: [2. Decisions and loops →](/guide-flow)*

## Installing Torvik

Torvik needs **clang** on your system — it's the tool Torvik hands its finished machine code
to for the final link step. Most Linux systems either have it or install it in one command
(`sudo apt install clang`, `sudo dnf install clang`, and so on). On Windows, install
[LLVM/clang with the MinGW-w64 toolchain](https://github.com/mstorsjo/llvm-mingw/releases).

Then install Torvik itself.

**Linux:**

```
curl -fsSL https://raw.githubusercontent.com/torvik-lang/torvik/main/linux/install.sh | sh
```

**Windows (PowerShell):**

```
iwr -useb https://raw.githubusercontent.com/torvik-lang/torvik/main/windows/install.ps1 | iex
```

Open a **new** terminal afterwards so your `PATH` picks up the change, then check it:

```
torvc version
rune version
```

`torvc` is the compiler. `rune` is the project tool — it creates projects, builds them, and
keeps your installation up to date. We'll use `torvc` directly for a while because it's the
simplest thing that works, and pick up `rune` in [Chapter 7](/guide-projects).

## Your first program

Create a file called `hello.tv` with exactly this in it:

```torvik
df main() -> void {
    echo!("Hello from Torvik!");
}
```

Compile it and run it:

```
torvc hello.tv -o hello
./hello
```

On Windows the last line is `.\hello.exe`. You should see:

```
Hello from Torvik!
```

That's a real native executable. You can copy it to another machine of the same kind and it
runs with nothing else installed.

### Reading that program line by line

Four things are happening, and all four matter.

**`df main()`** — `df` means "define a function." A **function** is a named block of
instructions. `main` is a special name: it's the **entry point**, the function the operating
system calls when your program starts. Every Torvik program has exactly one. The empty
parentheses `()` mean it takes no input.

**`-> void`** — this says what the function hands back when it finishes. `void` means
"nothing." Later you'll write functions that return a number or a piece of text.

**`{ ... }`** — curly braces group statements into a block. Everything between them is the
body of `main`.

**`echo!("Hello from Torvik!");`** — `echo!` prints a line of text. The text sits in double
quotes. The **semicolon** ends the statement, the way a period ends a sentence. Torvik needs
it, and forgetting it is a normal, clean compile error rather than a mystery.

### When you get it wrong

Delete the semicolon and compile again. You'll see something like:

```
error: expected a semicolon after this statement
  --> hello.tv:2:34
  |
2 |     echo!("Hello from Torvik!")
  |                                ^
```

The file, the line, the column, a caret under the exact spot. This is how every Torvik error
reads. Getting comfortable reading them now will save you hours later — they are usually
telling you precisely what's wrong, not hinting at it.

## Comments

A **comment** is a note for humans. The compiler ignores it completely.

```torvik
// A line comment: from the slashes to the end of the line.

set x: i64 = 1; // Comments can trail a statement too.

#- A block comment. -#

#-
   Block comments span as many lines as you like.
   #- And they nest, so you can comment out code
      that already contains block comments. -#
-#
```

Line comments start with `//`. Block comments open with `#-` and close with `-#`, and they
**nest** — which sounds like a footnote until the first time you try to comment out a chunk
of code that already has a block comment inside it, and it just works.

Comment *why*, not *what*. `x += 1; // add one to x` tells the reader nothing they can't see.
`x += 1; // header row is not data` tells them something real.

## Variables: `set` and `fixed`

A **variable** is a name for a value you want to keep. Torvik has two kinds:

- **`set`** creates a variable you can change later.
- **`fixed`** creates one you cannot. Attempting to reassign it is a compile error.

Both need a **type annotation** — you say up front what kind of value the name holds.

```torvik
df main() -> void {
    set count: i64 = 0;          // a whole number, changeable
    fixed name: str = "Sigrid";  // a piece of text, permanent

    count = count + 1;           // fine
    count += 1;                  // shorthand for the same thing

    // name = "Bjorn";           // compile error: name is fixed
    echo!("{name} has {count}");
}
```

The shape is always:

```
set   NAME: TYPE = VALUE;
fixed NAME: TYPE = VALUE;
```

**Reach for `fixed` by default.** Use `set` only when a value genuinely needs to change. A
`fixed` binding is a promise to everyone reading the code — including you in six months —
that this value is settled, and the compiler enforces it. Most values in a well-written
program never change after they're first assigned.

### Naming

Use lowercase words joined by underscores: `user_count`, `total_price`, `is_ready`. Names
should say what the value *is*. `n` is fine for a loop counter; `d` is not fine for
"days remaining."

## Types

A **type** is the kind of value a name holds. Torvik checks every one of them at compile
time.

### Whole numbers (integers)

Torvik has integer types in several sizes. The size controls how large a number it can hold
and how much memory it uses.

**Signed** (can be negative): `i8`, `i16`, `i32`, `i64`
**Unsigned** (zero or positive only): `u8`, `u16`, `u32`, `u64`
**Wide**: `i128`, `u128`

```torvik
set a: i64 = 42;
set small: u8 = 255;                        // 0 to 255
set huge: i128 = 100000000000000000000;     // far beyond i64's range
```

**Use `i64` unless you have a reason not to.** It's the everyday integer: it holds values up
to about 9.2 quintillion, which covers essentially everything. The smaller types exist for
when you're packing data tightly or matching an external format; the 128-bit types for when
you genuinely outgrow 64 bits.

A leading `-` negates a value: `-5`, or `set n: i64 = -x;`.

### Decimals

`f64` is a 64-bit floating-point number — a decimal.

```torvik
fixed pi: f64 = 3.14159;
echo!(pi);       // 3.14159
echo!(2.0);      // 2.0 - floats always print with a decimal point
```

One thing to know early, and it's true in every language, not just Torvik: floating-point
numbers are *approximations*. `0.1 + 0.2` is not exactly `0.3` in binary floating point.
This is fine for measurements and averages. For money, work in whole cents with an integer
instead.

### True and false

`bool` holds exactly one of two values: `true` or `false`.

```torvik
fixed ready: bool = true;
fixed empty: bool = false;
```

Booleans are what conditions produce, and they're the foundation of every decision your
program makes.

### Text

`str` holds text — a "string" of characters. String literals use double quotes.

```torvik
fixed greeting: str = "hello";
fixed letter: str = 'A';        // single quotes: a one-character string
```

Single quotes give you a **character literal**, which in Torvik is simply a one-character
string.

Two builtins bridge text and numbers:

```torvik
echo!(char_at("hello", 1));   // "e"  - the character at index 1, as a string
echo!(byte_at("hello", 1));   // 101  - its raw numeric byte value
echo!(chr(65));               // "A"  - the character for byte value 65
```

**Indexes start at 0.** The first character is at index `0`, the second at `1`. This trips up
everyone at first and then becomes second nature.

### Collections, briefly

`list<T>`, `table<K, V>`, and `bag<T>` hold many values at once. They get a
[whole chapter](/guide-collections).

## Operators

Operators combine values.

**Arithmetic:** `+` `-` `*` `/` `%`

```torvik
echo!(7 + 3);    // 10
echo!(7 - 3);    // 4
echo!(7 * 3);    // 21
echo!(7 / 3);    // 2  - integer division truncates toward zero
echo!(7 % 3);    // 1  - the remainder
```

Integer division throws away the fractional part: `7 / 3` is `2`, not `2.33`. If you want the
decimal, use `f64` values. The `%` operator ("modulo") gives the remainder, and it's more
useful than it looks — `n % 2 == 0` tests whether `n` is even.

**Comparison:** `==` `!=` `<` `>` `<=` `>=` — these produce a `bool`.

```torvik
echo!(5 == 5);   // true
echo!(5 != 5);   // false
echo!(3 < 5);    // true
```

Note `==` (two equals signs) *compares*, while `=` (one) *assigns*. Mixing them up is a rite
of passage.

**Logical:** `&&` (and), `||` (or), `!` (not)

```torvik
fixed a: bool = true;
fixed b: bool = false;

echo!(a && b);        // false - both must be true
echo!(a || b);        // true  - at least one must be true
echo!(!a);            // false - flips it
fixed all: bool = a && !b;   // combine freely
```

These **short-circuit**: in `a && b`, if `a` is false, `b` is never evaluated at all.

**Bitwise:** `&` `|` `^` `~` `<<` `>>` operate on the individual binary digits of integers.
You can ignore these until you need them — flags, low-level formats, and hashing are the
usual reasons.

```torvik
echo!(6 & 3);    // 2
echo!(1 << 3);   // 8  - shift left three places: 1 becomes 8
```

**Compound assignment:** `+=` `-=` `*=` `/=` `%=` update a variable in place.

```torvik
set x: i64 = 10;
x += 5;    // x is now 15; same as x = x + 5
x *= 2;    // 30
```

### Precedence

Multiplication and division bind tighter than addition and subtraction, as in ordinary
arithmetic; arithmetic binds tighter than comparison.

```torvik
echo!(2 + 3 * 4);      // 14, not 20
echo!((2 + 3) * 4);    // 20
```

**Use parentheses whenever they make the intent clearer.** Nobody has ever complained about
a parenthesis that made a line obvious.

## Printing and interpolation

`echo!` prints with a newline at the end. `echo` prints without one.

```torvik
echo("Loading... ");
echo!("done.");        // both appear on one line
```

`echo` handles any printable scalar: strings, every integer width, `f64`, and `bool`.

### Putting values into text

This is where Torvik is nicer than most languages. You don't need a formatting function —
`echo` does **interpolation** directly. Any `{name}` inside the string is replaced with that
variable's value, formatted correctly for its type.

```torvik
fixed name: str = "Odin";
set count: i64 = 3;
fixed ratio: f64 = 0.5;
fixed ok: bool = true;

echo!("Hello {name}!");              // Hello Odin!
echo!("{name} has {count} runes");   // Odin has 3 runes
echo!("ratio={ratio}, ok={ok}");     // ratio=0.5, ok=true
```

The braces can hold a **full expression**, not just a name:

```torvik
echo!("total: {price * qty}");    // arithmetic
echo!("doubled: {dbl(count)}");   // a function call
echo!("first: {runes[0]}");       // an index
```

For a literal brace, double it: `{{` and `}}`.

`echo` also takes several comma-separated arguments, printed in order with spaces between:

```torvik
echo!("count:", count);            // count: 3
echo!(name, "rolled", count);      // Odin rolled 3
```

### Building a string instead of printing one

When you want the *text itself* as a value — to store it, pass it, or return it — use `fmt`.
It interpolates exactly like `echo`, but hands back a `str`:

```torvik
fixed who: str = "World";
set n: i64 = 3;
fixed line: str = fmt("Hi {who}, n={n}");   // a str you can keep
```

`fmt` also supports **positional** placeholders — empty `{}` filled from the trailing
arguments, left to right:

```torvik
fixed label: str = fmt("{} + {} = {}", 2, 3, 5);   // "2 + 3 = 5"
```

In short: `echo` when printing, `fmt` when you need the string as a value or want positional
holes.

### Everywhere else, a string is just a string

Interpolation applies **only** to a string literal written directly as an argument to `echo`,
`echo!`, or `fmt`. Anywhere else, braces are ordinary characters:

```torvik
fixed css: str = ".a{color:red}";   // exactly that text, untouched
fixed slot: str = "{{title}}";      // exactly {{title}}
echo!(slot);                        // prints {{title}}
```

That last line is the important one: interpolation happens at **compile time on literals**,
never at run time on values. A variable that happens to contain braces prints them exactly as
they are. This is what lets CSS, JSON, and HTML templates sit in ordinary strings with no
escaping.

## Putting it together

A program using everything from this chapter:

```torvik
df main() -> void {
    fixed name: str = "Astrid";
    fixed price: i64 = 250;
    fixed qty: i64 = 3;

    set total: i64 = price * qty;
    total += 50;                          // shipping

    fixed free_shipping: bool = total > 500;

    echo!("Customer: {name}");
    echo!("{qty} items at {price} each");
    echo!("Total: {total}");
    echo!("Free shipping: {free_shipping}");
}
```

Save it, compile it, run it. Then change the numbers and run it again.

## Things to try

1. Print your name and age using interpolation in a single `echo!`.
2. Compute how many minutes are in a week, printing the result with a label.
3. Make a `fixed` variable and try to reassign it. Read the error carefully.
4. Work out `17 / 5` and `17 % 5` on paper, then check yourself with a program.
5. Build a string with `fmt` using positional `{}` placeholders and print it.
6. Print a literal `{hello}` — braces and all — without interpolation kicking in. (Two ways:
   escape them, or put the text in a variable first.)

---

*[← Guide index](/guide) · Next: [2. Decisions and loops →](/guide-flow)*
