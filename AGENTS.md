# Aspen Suites Guest Guide — Repository Guidelines

A small static website for guests of Aspen Suites: where to eat, what to do,
where the pool is, how the Wi-Fi works. No backend, no database, no login.
Content changes rarely. Later we may add paid listings for restaurants that
want to advertise; those are ordinary pages with `sponsored = true`.

## Stack (decided — do not add frameworks or services without asking)

- **Hugo** (extended, v0.165+) builds Markdown into HTML. No theme dependency:
  the handful of templates in `layouts/` are ours and are meant to stay small.
- **Plain CSS** in `static/css/site.css`, plus the Fraunces webfont from
  Google Fonts for headings. No JavaScript unless a page really needs it (an
  embedded map, for example). No Tailwind, no bundler, no npm.
- **GitHub Pages** hosts the site, deployed by `.github/workflows/hugo.yml`
  on every push to `main`. No domain yet; `baseURL` in `hugo.toml` is a
  placeholder and the workflow overrides it with the Pages URL at build time.
- **Beads (`bd`)** tracks work. There is no remote repo yet.

## Structure

| Path                        | What it is                                                        |
| --------------------------- | ----------------------------------------------------------------- |
| `content/`                  | All pages, as Markdown. One folder per section (see below).       |
| `content/<section>/_index.md` | Section title and intro shown on the list page.                 |
| `archetypes/place.md`       | Template for a restaurant / attraction / amenity page.            |
| `archetypes/default.md`     | Template for a plain text page.                                   |
| `layouts/`                  | HTML templates: `baseof`, `index` (home), `list`, `single`, partials. |
| `static/css/site.css`       | The stylesheet. Colours are CSS variables at the top.             |
| `static/`                   | Images and other files copied to the site as-is.                  |
| `hugo.toml`                 | Languages, site title, description, navigation menus per language. |
| `i18n/en.toml`, `i18n/bg.toml` | Interface strings (labels, footer text). Content is not here.   |
| `.github/workflows/hugo.yml`| Build + deploy to GitHub Pages.                                   |
| `public/`                   | Build output. Git-ignored; never edit.                            |

Sections (each is a folder under `content/` and an entry in `[[menus.main]]`):
`eat` (restaurants, cafés, bars), `do` (activities, attractions), `amenities`
(pool, gym, laundry, on-site things), `essentials` (Wi-Fi, check-out,
parking, pharmacy, emergency numbers). Add a section by creating the folder
with an `_index.md` and a menu entry in `hugo.toml`.

## Adding or editing content (the everyday workflow)

1. Create a page from the template:
   ```sh
   hugo new eat/luigis-pizza.md -k place      # restaurant, shop, trail, amenity…
   hugo new eat/luigis-pizza.bg.md -k place   # its Bulgarian translation
   hugo new essentials/check-out.md            # plain text page
   ```
   The file name becomes the URL (`/eat/luigis-pizza/`). Use lowercase and hyphens.
2. Open the file. Fill in the front matter (the block between `+++`). Delete
   fields you don't know; empty fields are simply not shown. Write the body
   in Markdown below it.
3. Set `draft = false` when it is ready. Drafts never appear on the live site.
4. Preview locally: `hugo server -D` and open http://localhost:1313 (`-D`
   shows drafts too).
5. Commit and push to `main`. GitHub Actions builds and publishes in ~1 minute.

Front matter fields for a place: `title`, `summary` (one sentence, shown in
lists), `weight` (lower = earlier in the list), `tags`, and under `[params]`:
`address`, `distance`, `phone`, `website`, `hours`, `price`, `map`,
`sponsored`. Order in lists is by `weight`, so give pages 10, 20, 30… to
leave room.

Images go in `static/images/` and are referenced as `![alt](/images/name.jpg)`.
Keep them under ~300 KB; resize before committing.

## Languages

English (default, at the site root) and Bulgarian (under `/bg/`). A page is
translated by adding a sibling file with the language code before `.md`:
`content/eat/luigis-pizza.md` (English) and `content/eat/luigis-pizza.bg.md`
(Bulgarian). Same front matter fields, translated values. A page without a
translation simply does not appear in the other language's lists, and the
header toggle only offers languages the current page exists in. Section
titles live in `_index.<lang>.md`; menu labels and the site title per
language are in `hugo.toml`; short interface labels (Distance, Hours,
Sponsored, the footer note) are in `i18n/<lang>.toml`. When adding an
English page, add the Bulgarian one in the same commit whenever you can.

To add a third language: copy a `[languages.xx]` block in `hugo.toml`, add
`i18n/xx.toml`, and translate `content/_index.xx.md` plus each section's
`_index.xx.md`.

## Commands

```sh
hugo server -D        # live preview with drafts
hugo                  # production build into public/
hugo --minify         # what CI runs
```

## Look and feel

The design is ours, not a downloaded theme. Palette: pine green, alpine sky
blue, snow white, slate grey; amber is reserved for the Sponsored badge.
Headings in Fraunces (serif), body in the system sans. Page grid is 72rem
wide with the brand left and navigation right in the header bar; running text
is capped at 44rem. Home sections and list entries are white tiles in a grid;
on a place page the facts box sits to the right of the text on wide screens. The header photo (Gergiyski lakes, Pirin, in
`static/images/pirin-lakes.jpg` with a 900px copy for phones) is the one
decorative element: tall with the welcome text on the home page, a short band
on every other page, always under a dark scrim so the white header text reads.
Everything below it stays quiet: flat tiles with a hairline border, no
shadows, no animation. To change the photo, replace both files (keep them
under ~800 KB and ~200 KB) and update `photoCredit` in `hugo.toml`. To
recolour, edit the variables at the top of `site.css`. The 7 MB original photo
in the repo root is git-ignored; never commit originals.

## Style

- Write for a guest on a phone: short pages, plain words, the useful facts
  first (distance, hours, price). One place per page.
- Keep templates readable over clever. If a layout change needs more than a
  few lines of Hugo templating, stop and consider whether the content can
  carry it instead.
- Facts about third parties (hours, prices) drift. Prefer "check the website"
  links over copying menus and prices in detail.
- Sponsored listings must show the `Sponsored` badge (set `sponsored = true`);
  never hide it.
- No secrets, keys or personal guest data in the repo. It is a public site.

## Analytics

Google Analytics is wired in `layouts/_default/baseof.html` via Hugo's
built-in template and switched on by setting `services.googleAnalytics.id`
in `hugo.toml`. It only loads in production builds. Guests are mostly in the
EU, so before enabling it decide on cookie consent (GA sets cookies) or use a
cookieless tool instead; see the analytics bead.

## Commits

Short imperative subject lines (`Add Luigi's Pizza`, `Fix pool hours`).
Content-only changes can be committed straight to `main`. Template or
workflow changes: build locally first (`hugo`) and check one page of each
kind renders. Do not push without being asked; there is no remote yet.

## Work tracking (beads)

Use `bd` for anything that outlives the current session: content still to
gather, template ideas, launch tasks. `bd ready` shows what can be started;
`bd create "…"`, `bd update <id> --status in_progress`, `bd close <id>`.
Operational gotchas go to `bd remember`, not into this file. Conventions and
decisions go into this file. Keep both current: when you change how
something works, update AGENTS.md in the same commit.
