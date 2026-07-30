---
title: Guide 4 - Collections and text
description: Store many values with lists, tables, and bags, test membership, and work with strings using Torvik's full set of text builtins.
---

# 4. Collections and text

*[← 3. Functions](/guide-functions) · [Guide index](/guide) · Next: [5. Errors and safety →](/guide-errors)*

So far every variable has held **one** value. Real programs handle many at once — a list of
users, a count per word, a set of things already seen. Torvik has three collections for this,
plus a full set of text operations.

## Lists: `list<T>`

A **list** is an ordered, growable sequence. The `<T>` says what it holds: `list<i64>` is a
list of integers, `list<str>` a list of strings.

```torvik
df main() -> void {
    set xs: list<i64> = list_new();

    push(xs, 10);
    push(xs, 20);
    push(xs, 30);

    echo!(xs[0]);        // 10 - first element
    echo!(xs[2]);        // 30 - third element
    echo!(len(xs));      // 3
}
```

- `list_new()` creates an empty list.
- `push(list, value)` appends to the end.
- `list[i]` reads the element at index `i` — **counting from 0**.
- `len(list)` gives the number of elements.

A list of three items has indexes 0, 1, and 2. Index 3 is past the end, and reading it stops
the program with a clear out-of-bounds message rather than handing back garbage.

### Modifying a list

```torvik
set xs: list<i64> = list_new();
push(xs, 10); push(xs, 20); push(xs, 30);

list_insert(xs, 1, 15);            // insert 15 at index 1 -> [10, 15, 20, 30]
list_remove(xs, 0);                // remove index 0       -> [15, 20, 30]
fixed last: i64 = list_pop(xs);    // remove and return the last -> 30

xs[0] = 99;                        // assign to an existing index
```

### Walking a list

Iterate directly with `each` — the clearest form:

```torvik
set words: list<str> = list_new();
push(words, "for"); push(words, "the"); push(words, "horde");

each w in words {
    echo!(w);
}
```

When you need the index too, range over the length:

```torvik
each i in 0..len(xs) {
    echo!("{i}: {xs[i]}");
}
```

`0..len(xs)` is exactly right: exclusive at the end, so it covers 0 through the last valid
index and never runs off.

Accumulating is the other everyday pattern:

```torvik
set total: i64 = 0;
each v in xs {
    total += v;
}
echo!("sum: {total}");
```

### Element types and inference

Lists hold any element type — `list<str>`, `list<f64>`, `list<bool>`, `list<i128>`. The type
usually comes from the annotation, but Torvik can also **infer** it from the first `push`:

```torvik
set names = list_new();      // element type not stated...
push(names, "odin");         // ...inferred as str from this push
echo!(names[0]);             // odin
```

Inference covers `str`, integer, `f64`, and `bool` elements. Annotate when there's no visible
`push`, or for other element types. Writing the annotation is never wrong, and it documents
intent — prefer it in code others will read.

## Tables: `table<K, V>`

A **table** maps keys to values — a phone book rather than a numbered row of lockers. Look
things up by name instead of position.

```torvik
df main() -> void {
    set ages: table<str, i64> = table_new();

    table_set(ages, "Ivar", 30);
    table_set(ages, "Freya", 27);

    echo!(table_get(ages, "Ivar"));    // 30
    echo!(table_has(ages, "Ivar"));    // true
    echo!(table_len(ages));            // 2

    table_del(ages, "Ivar");
    echo!(table_has(ages, "Ivar"));    // false
}
```

Looking up a missing key returns a **zero value** — `0` for an integer table. Use
`table_has` when "absent" and "zero" mean different things:

```torvik
check table_has(scores, name) {
    echo!("{name}: {table_get(scores, name)}");
} fallback {
    echo!("{name} has no score yet");
}
```

Setting a key that already exists replaces its value, so tables also work as counters:

```torvik
df count_words(words: list<str>) -> table<str, i64> {
    set counts: table<str, i64> = table_new();
    each w in words {
        set current: i64 = table_get(counts, w);   // 0 if absent
        table_set(counts, w, current + 1);
    }
    return counts;
}
```

### Iterating a table

Take the keys with `table_keys`, which returns them as a **sorted** `list<str>`, then fetch
each value:

```torvik
fixed ks: list<str> = table_keys(ages);
each k in ks {
    echo!("{k} is {table_get(ages, k)}");
}
```

They're sorted deliberately: a hash table's internal order shifts as it grows, so iterating
raw storage would give you a different order run to run. Sorted keys mean **reproducible
output** — the same input produces the same report every time.

## Bags: `bag<T>`

A **bag** is a set-like collection for membership questions — "have I seen this before?"

```torvik
set seen: bag<str> = bag_new();

bag_add(seen, "rune");
bag_add(seen, "rune");           // adding again is harmless

echo!(bag_has(seen, "rune"));    // true
echo!(bag_len(seen));            // 1

bag_remove(seen, "rune");
```

Reach for a bag when you only care about presence:

```torvik
df first_repeat(words: list<str>) -> str {
    set seen: bag<str> = bag_new();
    each w in words {
        check bag_has(seen, w) { return w; }
        bag_add(seen, w);
    }
    return "";
}
```

### Choosing a collection

| Question | Use |
|---|---|
| Order matters, or I need position | `list<T>` |
| I look things up by a key | `table<K, V>` |
| I only ask "is it in there?" | `bag<T>` |

## Membership: `<|`

The membership operator tests whether a value is in a collection, giving a `bool`:

```torvik
set ids: list<i64> = list_new();
push(ids, 10); push(ids, 20); push(ids, 30);

check 20 <| ids {
    echo!("present");
}

fixed known: bool = name <| valid_names;
```

Anything can go on the left — a variable, a literal, or a call result. Integer lists compare
by value and **string lists compare by content**, so a freshly built string matches an equal
stored one. A type mismatch between the item and the list's elements is a compile error.

For a large collection checked repeatedly, a `bag` is faster than scanning a list; `<|` on a
list examines elements in turn.

## Working with text

Strings get their own toolkit. Every one of these returns a **new** string — Torvik strings
are values, and operations never modify in place.

### Length and pieces

```torvik
echo!(len("hello"));               // 5
echo!(substr("hello", 0, 4));      // "hell" - start..end, end exclusive
echo!(char_at("hello", 1));        // "e"
echo!(byte_at("hello", 1));        // 101
echo!(chr(65));                    // "A"
```

`substr(s, start, end)` takes from `start` **up to but not including** `end` — the same
convention as ranges and indexes, so lengths line up: `substr(s, 0, 4)` gives four
characters.

### Joining

```torvik
echo!(str_concat("Hello, ", "Torvik"));         // Hello, Torvik
echo!(str_concat(key, " = ", value, "\n"));     // any number of parts
```

`str_concat` takes two or more arguments (one is a compile error), so you never need to nest
it. For anything with values interleaved, `fmt` usually reads better:

```torvik
fixed line: str = fmt("{key} = {value}");
```

### Trimming and case

```torvik
echo!(trim("  hi  "));     // "hi"
echo!(triml("  hi  "));    // "hi  " - left only
echo!(trimr("  hi  "));    // "  hi" - right only
echo!(upper("hi"));        // "HI"
echo!(lower("HI"));        // "hi"
```

`trim` on user input is almost always right — a trailing space is invisible and breaks
comparisons.

### Searching and testing

```torvik
echo!(contains("hello", "ell"));    // true
echo!(starts("hello", "he"));       // true
echo!(ends("hello", "lo"));         // true
echo!(find("hello", "l"));          // 2 - first index, or -1 if absent
```

Since v1.4.0 these compare directly against a literal:

```torvik
check contains(line, "error") == false {
    echo!("clean line");
}
```

### Replacing and splitting

```torvik
echo!(replace("a.b.c", ".", "/"));            // a/b/c

set parts: list<str> = split("x,y,z", ",");   // ["x", "y", "z"]
each p in parts { echo!(p); }
```

`split` is how you parse almost anything line-oriented: split a file on `"\n"`, split each
line on `","` or `"\t"`, and work with the pieces.

### Comparing strings

Strings compare **by content**, never by identity:

```torvik
check name == "Astrid" { echo!("match"); }
check "apple" < "banana" { echo!("sorted"); }     // lexicographic
```

`<`, `<=`, `>`, `>=` order strings lexicographically (byte order), which is what you want for
sorting. Comparing a string with a number is a clean compile error — convert first.

### Converting

```torvik
fixed n: i64 = toint("42");
fixed f: f64 = tofloat("3.14");
fixed s: str = tostr(42);
```

One practical note: bind a conversion result to a variable before using it in a larger
arithmetic expression.

```torvik
set n: i64 = toint("7");
echo!(n + 1);              // 8

// echo!(toint("7") + 1);  // compile error - bind it first
```

It's a clean compile error, never a wrong answer. And if the text might not be a valid
number, use `try_toint` instead, which reports failure properly — that's
[the next chapter](/guide-errors).

## Putting it together

A word-frequency report using all three collections:

```torvik
df main() -> void {
    fixed text: str = "the raven flew over the hall and the raven landed";

    fixed words: list<str> = split(text, " ");

    set counts: table<str, i64> = table_new();
    set seen: bag<str> = bag_new();
    set repeats: list<str> = list_new();

    each w in words {
        fixed clean: str = w ~> trim ~> lower;

        set current: i64 = table_get(counts, clean);
        table_set(counts, clean, current + 1);

        check bag_has(seen, clean) {
            check clean <| repeats {
                // already recorded
            } fallback {
                push(repeats, clean);
            }
        } fallback {
            bag_add(seen, clean);
        }
    }

    echo!("{len(words)} words, {table_len(counts)} unique");
    echo!("");

    fixed ks: list<str> = table_keys(counts);
    each k in ks {
        echo!("  {k}: {table_get(counts, k)}");
    }

    echo!("");
    echo!("repeated words:");
    each r in repeats {
        echo!("  {r}");
    }
}
```

Because `table_keys` sorts, that report is identical every run.

## Things to try

1. Build a `list<i64>` of ten numbers and print the largest, the smallest, and the average.
2. Reverse a list into a new list. Then reverse a **string** using `char_at` and
   `str_concat`.
3. Count the vowels in a sentence using a `bag<str>` of vowels and `<|` or `bag_has`.
4. Build a `table<str, str>` phone book, then print it sorted by name.
5. Split a CSV line like `"name,age,city"` and print each field on its own line, numbered.
6. Write `is_palindrome(s: str) -> bool` that ignores case and spaces.
7. Take the word-frequency program and make it print only words appearing more than once.

---

*[← 3. Functions](/guide-functions) · [Guide index](/guide) · Next: [5. Errors and safety →](/guide-errors)*
