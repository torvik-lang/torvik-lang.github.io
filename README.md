# torvik-lang.github.io

The Torvik website - https://torvik-lang.github.io - woven with
[Vefna](https://github.com/torvik-lang/vefna), the static site generator written in
Torvik.

## Layout

- `vefna.site`, `content/`, `templates/`, `static/` - the Vefna site source
- `docs/` - the built site, served by GitHub Pages (Settings -> Pages -> deploy from
  branch `main`, folder `/docs`)

## Updating the site

1. Edit the Markdown in `content/` (or the template / stylesheet).
2. Rebuild and refresh `docs/`:

   ```
   vefna build --clean
   cp -r site/. docs/
   ```

   (`docs/.nojekyll` is committed and must stay - it tells GitHub Pages to serve the
   files as-is.)
3. Upload the changed files in `docs/` (plus any changed source files) to `main`.
