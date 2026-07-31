---
title: Guide 8 - Systems and OS development
description: Talk to hardware directly and build a kernel that boots on bare metal - raw pointers, value structs, volatile memory, inline assembly, and freestanding builds, ending with a working x86_64 kernel you can run in QEMU.
---

# 8. Systems and OS development

*[← 7. Modules and projects](/guide-projects) · [Guide index](/guide) · Next: [Appendix: reference →](/guide-reference)*

Everything up to now has assumed an operating system underneath you. When you called
`writefile`, Linux or Windows did the actual writing. When your program ended, something
cleaned up after it.

This chapter is about what happens when there is nothing underneath. No operating system.
No C library. No Torvik runtime unless you ask for one. Just your code and the machine.

By the end you will have built a kernel — a program that boots on bare hardware — and you
will understand every line of it.

## The bargain

Talking to hardware means writing to addresses the machine cares about, and there is no way
to make that safe. A wrong address doesn't throw an exception; it corrupts something or
resets the computer.

Torvik's answer is not to make it safe. It is to make it **obvious**:

```torvik
unsafe set vga: varda<u16> = from_addr(0xB8000);
unsafe store_vol(vga, 0x0F48);
```

Every operation that can reach an arbitrary address requires the word `unsafe` on that
statement. Not a compiler flag, not a file-level switch, not a block you open once and
forget — the word, every time.

This matters more than it sounds. In a codebase with ten thousand lines and forty of them
unsafe, you can find those forty with a text search. The safe surface stays exactly as safe
as it was: `buf[i]` in the function next door is still bounds-checked, and reference counting
still cleans up after your strings.

> **A note on nerves.** If this chapter feels like a big step, that's reasonable — it *is*
> a different kind of programming. But nothing here is magic, and the compiler still tells
> you when you've got it wrong. Read it once for the shape of things; you don't need to
> retain the details.

---

## Numbers the way hardware writes them

Hardware documentation is written in hexadecimal and binary, so Torvik reads both:

```torvik
fixed vga:   i64 = 0xB8000;        // hexadecimal
fixed mask:  i64 = 0b1010_0110;    // binary
fixed big:   i64 = 1_000_000;      // underscores group digits
```

The underscores are ignored by the compiler. They exist so that `0b1010_0110` is readable
as four-plus-four bits, the way the manual prints it.

---

## Raw pointers: `varda<T>`

A **`varda<T>`** is a pointer to a `T`. No ownership, no reference counting — the machine's
own idea of a pointer, and nothing more.

```torvik
unsafe set p: varda<u16> = from_addr(0xB8000);
```

`from_addr` turns a number into a pointer. `as_addr` turns one back into a number. To read
and write through it:

```torvik
unsafe store(p, 0x0F48);
unsafe fixed value: i64 = load(p);
```

### Moving a pointer

Two operations, and the difference matters:

```torvik
unsafe set next: varda<u16> = ptr_add(p, 1);          // + 1 ELEMENT  = 2 bytes here
unsafe set raw:  varda<u16> = ptr_byte_offset(p, 1);  // + 1 BYTE
```

`ptr_add` scales by the size of the element, the way `p + 1` does in C. `ptr_byte_offset`
moves by raw bytes regardless of type.

They have deliberately different names because mixing them up produces code that compiles,
runs, and writes to the wrong place. That is the worst category of bug: no crash, no error,
just wrong data appearing somewhere else in memory.

### The safe pointer operations

Not everything about pointers is dangerous. Three operations don't need `unsafe`:

```torvik
fixed buf: [u8; 16] = array_zero();
set p: varda<u8> = addr_of(buf);      // safe: you already own buf
set nothing: varda<u8> = null_addr(); // safe: a null pointer
check is_null(nothing) == 1 { ... }   // safe: checking one
```

`addr_of` is safe because it takes the address of something that already exists and that you
already own. It cannot invent an address out of a number — that's what `from_addr` is for,
and that one is unsafe.

---

## Fixed arrays: `[T; N]`

A **fixed array** is `N` elements stored inline — in the function's own stack frame, not on
the heap:

```torvik
fixed buf: [u8; 4] = array_zero();
buf[0] = 72;
buf[1] = 105;
echo!(buf[0]);        // 72
echo!(len(buf));      // 4 - known at compile time, costs nothing at run time
```

`array_zero()` fills it with zeros. You can also write the elements out:

```torvik
fixed primes: [i64; 4] = [2, 3, 5, 7];
```

The count must match the declaration exactly — if it doesn't, that's a compile error, not a
surprise at run time.

**Indexing is bounds-checked**, including against negatives:

```
[Torvik panic] array index 5 out of bounds (length 2)
```

That check costs one comparison, and it is the difference between a clear message and an
afternoon of confusion. If you genuinely need unchecked access, that's what `addr_of` plus
`ptr_add` is for — and it says `unsafe` when you do.

Fixed arrays hold machine scalars: the integer widths, `bool`, and the float widths.

---

## Value structs: `shape`

A **`shape`** groups fields into one value, stored inline with no heap allocation and no
reference counting:

```torvik
shape Point { x: i64, y: i64 }

set p: Point = Point { x: 10, y: 20 };
echo!(p.x + p.y);     // 30

p.x = 100;
echo!(p.x);           // 100
```

Any field you leave out of the literal is **zero**, not stack garbage:

```torvik
set q: Point = Point { x: 5 };
echo!(q.y);           // 0
```

### `packed`: matching the hardware exactly

By default the compiler may insert padding between fields so each one sits at an address the
processor likes. That's faster, and for ordinary data it's what you want.

Hardware doesn't care what the processor likes. A descriptor table entry is defined byte by
byte, and a padding byte in the wrong place makes the whole structure wrong. So:

```torvik
shape Mixed        { a: u8, b: i64, c: u8 }    // size 24 - padded
packed shape Tight { a: u8, b: i64, c: u8 }    // size 10 - exact
```

`packed` tells the compiler to insert nothing. The layout then matches a C struct or a
hardware register block byte for byte, which is what makes Torvik able to describe things
like a GDT entry:

```torvik
packed shape GdtEntry {
    limit_low: u16,
    base_low:  u16,
    base_mid:  u8,
    access:    u8,
    flags:     u8,
    base_high: u8
}
```

`size_of(GdtEntry)` reports **8**, exactly as the Intel manual says it should.

### Asking about layout

```torvik
echo!(size_of(i64));        // 8
echo!(align_of(i64));       // 8
echo!(size_of(GdtEntry));   // 8
```

`size_of` and `align_of` are resolved at compile time — they cost nothing at run time — and
they work on any scalar or shape. The answers come from LLVM, so they stay correct as
targets change rather than from a table someone has to remember to update.

You can also take the address of a single field:

```torvik
unsafe set px: varda<i64> = addr_of(p.x);
unsafe store(px, 99);
echo!(p.x);           // 99
```

> **What shapes don't hold yet.** Shapes and fixed arrays currently hold machine scalars.
> Heap-backed fields (`str`, `list`, `table`, `bag`, `i128`/`u128`), nested shapes, and
> passing a shape to a function are planned for a later release — they need reference
> counting threaded through struct copies, which is real work worth doing carefully.
> Unsupported types are refused with a clear error, never silently mis-stored.

---

## Volatile access: making the write actually happen

Here is a bug that will cost you a day if nobody warns you about it.

You write a value to a hardware register. The optimiser looks at your code, sees a write to
memory that nothing ever reads, concludes it has no effect, and **deletes it**. Your code is
correct. Your program does nothing.

The optimiser isn't wrong — for ordinary memory that reasoning is sound. It just doesn't
know that this particular address is a device.

```torvik
unsafe store_vol(reg, value);
unsafe fixed status: i64 = load_vol(reg);
```

`load_vol` and `store_vol` are **volatile**: never deleted, never merged with a neighbouring
access, never moved past another volatile one. When the value *is* the point — and for
hardware it always is — use these.

Rule of thumb: if the address refers to a device rather than to your own data, it's volatile.

---

## Inline assembly: `galdr`

Sometimes there is no way to say it in a language. `galdr` — the Old Norse word for a chanted
spell — drops you to assembly:

```torvik
unsafe galdr { "cli"; "hlt"; }
```

One instruction per string. The block is emitted with a marker that tells the optimiser it
has effects it can't see, so it stays exactly where you put it.

### Named intrinsics instead

Most of what you actually need has a name already, so you don't have to get register
constraints right:

```torvik
unsafe cli();                    // disable interrupts
unsafe sti();                    // enable interrupts
unsafe hlt();                    // stop until an interrupt
unsafe pause_cpu();              // spin-loop hint

unsafe outb(0x3D4, 14);          // write a byte to an I/O port
unsafe fixed v: i64 = inb(0x3D5); // read one back
```

`outb`/`outw`/`outl` and `inb`/`inw`/`inl` cover the three widths. These compile to exactly
the instruction you'd write by hand — `out %al,(%dx)` — with the constraint spelling living
in the compiler where it was written once and tested, rather than in every kernel.

### The two ways hardware talks

x86 has two separate mechanisms, and it's worth knowing which you're using:

- **Memory-mapped I/O** — the device pretends to be memory. You write with a volatile store
  to an address. The VGA text buffer at `0xB8000` works this way.
- **Port I/O** — a completely separate address space, reached only with the `in` and `out`
  instructions. The VGA *cursor* works this way.

Same chip, two different doors. The kernel below uses both.

---

## Freestanding builds

```sh
torvc kernel.tv --bare -o kernel.elf
```

`--bare` produces an image with no C library and no Torvik runtime. What you get is a static
ELF with an entry point and your code — nothing else.

Ask for something that needs an operating system and the compiler stops you at compile time,
rather than letting the linker fail with a wall of undefined symbols:

```
error: 'readfile' needs an operating system, and this is a freestanding build (--bare) -
there is nothing underneath to service it. Talk to the hardware instead: volatile stores
for memory-mapped I/O, outb/inb for ports. If you did want the OS, drop --bare.
```

File I/O, `args()`, `time_ms`, `sys_run` and `apply std` are all refused. What still works is
everything that doesn't need a kernel: arithmetic, `check`, `whilst`, functions, shapes,
arrays, pointers, and `size_of` — which is a compile-time fact and perfectly happy bare.

**The output is decided by the target, not by your machine.** A kernel built on Windows is
the same ELF as one built on Linux. That is deliberate: a kernel has no business caring what
laptop compiled it.

### Memory, if you want it

`str` and `list` need to allocate, and bare there is no `malloc`. Torvik doesn't pretend
otherwise — it asks you where memory comes from:

```torvik
df on_alloc(size: i64) -> varda<u8> { ... }
df on_free(p: varda<u8>) -> void { ... }
```

Define both and the runtime's allocation is wired to them, and heap types work in a
freestanding image. Leave them out and the runtime isn't linked at all, which is why a
minimal kernel is so small.

A **bump allocator** is a perfectly respectable choice here — hand out aligned blocks from a
region of memory and never reclaim any. A kernel that never frees doesn't need more:

```torvik
fixed ARENA_BASE: i64 = 0x200000;
fixed ARENA_SIZE: i64 = 0x100000;
set arena_off: i64 = 0;

df on_alloc(size: i64) -> varda<u8> {
    set n: i64 = size;
    fixed rem: i64 = n - ((n / 16) * 16);
    check rem != 0 { n = n + (16 - rem); }      // round up to 16 bytes
    check arena_off + n > ARENA_SIZE {
        unsafe set nul: varda<u8> = from_addr(0);
        return nul;                              // out of memory
    }
    fixed at: i64 = ARENA_BASE + arena_off;
    arena_off = arena_off + n;
    unsafe set block: varda<u8> = from_addr(at);
    return block;
}

df on_free(p: varda<u8>) -> void {
    // A bump allocator has nothing to give back.
}
```

Returning null when the arena runs out is the contract. The runtime panics, and bare, a
panic parks the machine — there is nowhere to report to.

---

## Building a kernel

Now we put it together.

### Step 1: the smallest thing that counts

```torvik
df main() -> void {
    unsafe set vga: varda<u16> = from_addr(0xB8000);
    unsafe store_vol(vga, 0x0F48);
    whilst true {
        unsafe hlt();
    }
}
```

```sh
torvc hello_kernel.tv --bare -o hello.elf
```

Each VGA cell is a `u16`: the low byte is the character, the high byte the colour. `0x0F48`
is `0x48` (`H`) in colour `0x0F` (white on black).

The `hlt` loop matters. `whilst true { }` would spin a core at 100% forever; `hlt` stops the
processor until an interrupt arrives, and since we never enable interrupts, it stops for
good — at no cost.

### Step 2: making it bootable

That image is freestanding, but a bootloader won't touch it yet: nothing identifies it as a
kernel. The fix is a **Multiboot header** — a magic number in the first 8 KiB that says "I am
a kernel, load me at 1 MiB, jump to my entry point".

It goes in the linker script, because it must sit at an exact place before any code:

```ld
ENTRY(_start)

SECTIONS
{
    . = 1M;

    .multiboot ALIGN(4) :
    {
        KEEP(*(.multiboot))
    }

    .text ALIGN(4K) : { *(.text) *(.text.*) }
    /* ... rodata, data, bss ... */
}
```

`. = 1M;` places the kernel at one megabyte. Everything below that is real-mode memory, the
VGA window, and firmware — not yours to use.

> **`KEEP` is not optional.** Nothing in your program references the Multiboot header, so
> the linker's dead-code elimination will happily discard it. The image links without a
> single warning and then never boots. `KEEP` is what stops that.

### Step 3: the part nobody warns you about

Here is the trap that catches every first kernel: **Multiboot hands control to you in 32-bit
protected mode.** Torvik compiles to 64-bit code. Jump straight from one to the other and
the processor misreads your first instruction and dies.

Something has to bridge that gap, and it cannot be written in Torvik — it runs *before* long
mode exists, so by definition it isn't 64-bit code. That's `boot.s`, and it is the only
assembly in the project:

1. Build page tables. Long mode refuses to start without paging already enabled.
2. Identity-map the first gigabyte with 2 MiB pages, so physical and virtual addresses are
   the same number and nothing beneath you moves.
3. Set `CR4.PAE`, then `EFER.LME`, then `CR0.PG` — in that order. Long mode is *armed* by the
   MSR and *engaged* by enabling paging.
4. Load a 64-bit GDT and far-jump into it. The far jump is what actually reloads `CS` and
   puts the processor in 64-bit mode.
5. `call kernel_main` — and from here on, you are writing Torvik.

Sixty lines, once, ever. The complete file is in
[`examples/kernel/boot.s`](https://github.com/torvik-lang/torvik/blob/main/examples/kernel/boot.s).

### Step 4: build and run

```sh
torvc kernel.tv --bare --elf32 --entry kernel_main \
      --link-script linker.ld --link-with boot.s -o kernel.elf

qemu-system-x86_64 -kernel kernel.elf
```

> **Why `--elf32`?** Multiboot 1 will not load a 64-bit ELF at all — QEMU tells you
> *"Cannot load x86-64 image, give a 32bit one."* The flag rewrites the finished image
> as a 32-bit ELF container. The **code inside stays 64-bit**; only the wrapper changes,
> which is safe because the link is already complete. Without it the image is still a
> valid freestanding ELF — it just won't boot.

`--link-with` assembles `boot.s` for the same target and links it alongside your code.
`--entry kernel_main` names the symbol torvc emits for your `main` — which is what `boot.s`
calls once the processor is in long mode. You still write `df main()` exactly as normal.

A window opens with text in it. That text came from a `.tv` file.

### Step 5: let rune do it

Four flags on every build gets old. Declare them once in `torvik.rune`:

```toml
[project]
name = mykernel
version = 0.1.0

[build]
target = bare
entry = kernel_main
arch = x86_64-unknown-none-elf
link-script = linker.ld
link-with = boot.s
elf32 = true
runner = qemu-system-x86_64 -kernel {output}
```

Then the ordinary commands do the right thing:

```sh
rune build      # -> build/mykernel.elf
rune run        # builds, then hands the image to QEMU
```

A bare project builds to `.elf` rather than the host's executable extension — calling a
kernel `mykernel.exe` would be a lie. And `rune run` doesn't try to *execute* it, because
your machine can't; it passes the image to whatever `runner` names.


### If it boots and then reboots forever

The single most common failure in a first x86_64 kernel, and worth recognising on
sight: the machine starts, prints a line or two, and resets — forever.

That is a **triple fault**. Something raised an exception, there is no interrupt
descriptor table to handle it, so it escalated to a double fault, which had no
handler either, and the processor gave up and reset.

The usual culprit is **SSE**. A compiler targeting x86_64 assumes SSE is available —
it is part of the base architecture — and emits SSE instructions freely, even for
something as ordinary as zeroing a small array. But the processor comes out of reset
with SSE *disabled*, and executing one raises `#UD`. `boot.s` enables it immediately
after entering long mode, by clearing `CR0.EM`, setting `CR0.MP`, and setting
`CR4.OSFXSR` and `CR4.OSXMMEXCPT`.

The tell is *where* it dies: always at the first line that touches a wide memory
operation, so the output stops at a consistent point rather than randomly.

---

## What the reference kernel does

The full kernel in [`examples/kernel/`](https://github.com/torvik-lang/torvik/tree/main/examples/kernel)
puts every piece of this chapter to work:

- **VGA text output** — a `vga_put` that computes a cell offset and writes it volatilely,
  a `vga_clear`, and a `vga_print` that walks a string.
- **Port I/O** — moving the hardware cursor through the CRTC index/data port pair, and
  reading a register back with `inb`.
- **A decimal printer** without allocation — digits extracted into a `[u8; 20]` and written
  straight to the screen.
- **The allocator hook**, so `str` works.
- **`size_of`**, displayed on screen to show that compile-time layout facts survive into a
  freestanding image.

It ends the way kernels do:

```torvik
unsafe cli();
whilst true {
    unsafe hlt();
}
```

---

## Where to go next

The classic path from here: a **GDT** of your own, then an **interrupt descriptor table** so
the keyboard can talk to you, then a **timer**, then **paging** and a real allocator behind
`on_alloc`.


---

## Exercises

1. Change the colour of the text. The high byte is `0xBF` — background `B`, foreground `F`.
2. Write a `vga_print_at(row, col, text)` and print your name in the middle of the screen.
3. Make a `packed shape` for a Multiboot header and check `size_of` reports 12.
4. Fill the screen with a colour gradient by looping over all 2000 cells.
5. Remove `KEEP` from the linker script and confirm the image still *links*. Then look at
   the section list and see that the header is gone.
6. Build the same kernel on both Linux and Windows and compare the two files.
7. Add a `panic_to_screen` that writes a message and halts, and call it from your allocator
   when the arena runs out.

---

*[← 7. Modules and projects](/guide-projects) · [Guide index](/guide) · Next: [Appendix: reference →](/guide-reference)*
