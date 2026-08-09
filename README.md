# DIY Hugo Scaffold

Shared Hugo/[Stack theme](https://github.com/CaiJimmy/hugo-theme-stack) site scaffolding
for the `jens-goes-mad` DIY-* GitHub Pages docs sites (`DIY-KRONOS-EDITOR`,
`DIY-MIDI-METRONOME.public`, `DIY-PEDALBOARD.public`, ...). Each of those sites used to
carry its own hand-copied version of this plumbing -- which drifted: a real bug fixed in
one site's copy (a local-dev-server `baseURL` trap that 404'd images not yet pushed to
production) was still silently present, unfixed, in the other two when this repo was
split out. Fixing it once here, instead of three times, is the entire point.

## What's actually in here, and how each part gets used

- **`layouts/`, `assets/scss/`, `assets/icons/`** -- imported into a consuming site as a
  real [Hugo Module](https://gohugo.io/hugo-modules/use-modules/), the exact same
  mechanism each site already uses to pull in the Stack theme itself. Hugo's default
  per-module mounts (`layouts`->`layouts`, `assets`->`assets`, etc.) apply automatically,
  no explicit `[[imports.mounts]]` needed on the consuming side.
- **`config/_default/{module.toml,permalinks.toml,markup.toml}`, `docker-compose.yml`**
  -- **reference copies only, not module-imported.** Hugo's own configuration loading
  only ever reads `config/` from the main project, never from an imported module -- there
  is no supported way to have a dependency module inject site config the way it can
  inject layouts/assets. These three files (and docker-compose.yml, which Hugo Modules
  doesn't touch at all -- it's plain dev tooling) are tiny and change rarely; copy them
  into a new site by hand, and re-copy if this repo's copy changes.

## Wiring up a consuming site

In the site's own `docs/go.mod` (or wherever its Hugo project root's `go.mod` lives),
add a second import alongside the theme -- **order matters**: this scaffold's layouts
intentionally override some of the theme's own (e.g. the homepage-redirect trick, the
right-sidebar partial), so it must be imported *after* the theme for its files to win:

```toml
# config/_default/module.toml
[[imports]]
path = "github.com/CaiJimmy/hugo-theme-stack/v3"

[[imports]]
path = "github.com/jens-goes-mad/DIY-HUGO-SCAFFOLD.public"
```

```
# go.mod
require (
    github.com/CaiJimmy/hugo-theme-stack/v3 v3.30.0 // indirect
    github.com/jens-goes-mad/DIY-HUGO-SCAFFOLD.public v0.0.0-... // pinned via `hugo mod get`
)
```

Then run `hugo mod get github.com/jens-goes-mad/DIY-HUGO-SCAFFOLD.public` (or the
project's `docker compose` Hugo image's equivalent) from the site's Hugo project root to
resolve and pin an actual version, and delete that site's own now-duplicate
`layouts/`/`assets/scss/`/`assets/icons/` (the imported module supplies them instead).
Keep the site's own `content/`, `config/_default/config.toml` (baseurl),
`config/_default/params.toml` (branding), `config/_default/menu.toml` (social links) --
those stay genuinely per-project.

## Updating a consuming site to a newer version of this repo

`hugo mod get -u github.com/jens-goes-mad/DIY-HUGO-SCAFFOLD.public` from the consuming
site's Hugo project root, then rebuild and spot-check before committing the updated
`go.sum` -- a fix landing here should be a deliberate pull per site, not something that
silently changes a site's build on its own.
