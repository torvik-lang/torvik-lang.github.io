---
title: Vefna
description: Vefna - a static site generator woven in Torvik. Markdown in, HTML out, in parallel.
---

# Vefna

**A static site generator woven in Torvik.** *(VEF-nah - coined from the Old Norse*
vefa*, "to weave," and* vefr*, "web.")*

Vefna reads a folder of Markdown, presses it through your HTML templates, and writes a
complete static site - ready for GitHub Pages, Cloudflare Pages, Netlify, or any plain
web server. No runtime, no database, no dependencies: one compiled binary.

It is also Torvik's flagship showcase: pages render in parallel on **eight raven worker
threads** fed by typed bridge channels - 500+ pages weave in well under a second - and
builds are incremental and fully deterministic. **This site is woven by Vefna.**

## Install

Linux:

```
curl -fsSL https://raw.githubusercontent.com/torvik-lang/vefna/main/linux/install.sh | sh
```

Windows (PowerShell):

```
iwr -useb https://raw.githubusercontent.com/torvik-lang/vefna/main/windows/install.ps1 | iex
```

Or grab a binary from the [releases page](https://github.com/torvik-lang/vefna/releases)
and put it on your PATH.

**Updating:** re-run the install line above - it always fetches the latest release.
Pin a version with `VEFNA_VERSION=1.1.0` (Linux) or `$env:VEFNA_VERSION = "1.1.0"`
(Windows) before running it; uninstall with the script's `--uninstall` flag.

## Quick start

```
vefna new myblog
cd myblog
vefna serve
```

`vefna serve` builds the site and hosts it at `http://127.0.0.1:8000`, rebuilding as
you write; `vefna build` writes the finished site to `site/` for you to push to any
static host.

## What you get

- A pragmatic **Markdown** subset: headings, paragraphs, bold, italic, inline code,
  links, images, lists, blockquotes, fenced code blocks, horizontal rules - all output
  HTML-escaped
- **Front matter** (`title`, `description`, `author`, `date`, `template`) with
  site-wide fallbacks from one `vefna.site` config
- Plain **HTML templates** with `{{slot}}` placeholders and per-page template selection
- **Incremental builds** - a page reweaves only when its source, a template, or the
  config is newer than its output
- **Deterministic output** - the same inputs produce a byte-identical site on every
  platform, every time
- `static/` assets copied binary-safe, folder structure carried straight through
- A built-in **preview server** (`vefna serve`) and **watch mode** (`vefna watch`) that
  rebuild as you edit, plus **draft pages** (`draft: true` front matter, shown only with
  `--drafts`)

Everything else - the full command reference, template slots, and hosting notes - is in
the [README](https://github.com/torvik-lang/vefna).

> The Norns weave the threads of fate. Vefna weaves only web pages, which is safer.
