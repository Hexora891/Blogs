# Kana Blog — Setup, Usage, and Customization Guide

This guide documents the Hugo blog in this directory and the
[Kana theme](https://github.com/ChrisWiegman/kana-hugo-theme) it uses. It is written to live
**outside** the site's publishing pipeline: it sits at the project root, not under `content/`, so
Hugo never renders or deploys it. Keep it wherever you like.

> Theme: Kana ("A warm, earthy dark theme") by Chris Wiegman — MIT licensed.
> Static site generator: Hugo (extended) — this project was built and tested with **Hugo v0.167.0**.

---

## 1. What was set up

- A complete Hugo site at the project root.
- The Kana theme installed as a **git submodule** at `themes/kana-hugo-theme`.
- A configured `hugo.toml` with menus, SEO, responsive images, and search enabled.
- Sample content: 3 blog posts, 2 notes, 3 library books, plus About / Uses / Now / Policies pages.
- A git repository initialised for the site (with a `.gitignore` for build output).
- Demo assets (header icon, favicon set, a few images) copied into `assets/` and `static/`.

The site builds cleanly and the dev server serves every page. See section 4 to run it.

---

## 2. Requirements

- **Hugo extended, v0.152.0 or newer.** The theme uses WebP image processing and SCSS, which require
  the *extended* build. A standard (non-extended) Hugo will fail to build.
  - This machine already has it at `~/.local/bin/hugo` (v0.167.0). Verify with:
    ```bash
    hugo version   # must say "+extended"
    ```
  - Install/upgrade on Arch: `sudo pacman -S hugo`
  - Any OS (official binaries): https://gohugo.io/installation/
- **Node.js + npm** — only needed for the theme's own Playwright tests and screenshot tooling. You do
  **not** need Node to build or run your site.

---

## 3. Project structure

```
blogs/
├── hugo.toml                 # ← main configuration (edit this first)
├── KANA-BLOG-GUIDE.md        # ← this file (not published)
├── .gitignore
├── archetypes/               # templates for `hugo new`
├── assets/                   # files processed by Hugo (SCSS, images to resize)
│   └── images/               # header icon + featured images for posts
├── content/                  # your writing (the only place pages come from)
│   ├── _index.md             # homepage hero content
│   ├── about.md  uses.md  now.md  policies.md
│   ├── blog.md               # the /blog archive page
│   ├── library.md            # the /library reading page
│   ├── search.md             # the /search page
│   ├── posts/                # blog posts  → section "posts"
│   │   └── 2026/*.md
│   ├── notes/                # short-form notes → section "notes"
│   │   ├── _index.md
│   │   └── 2026/*.md
│   └── books/                # headless bundle for the library
│       ├── index.md          #   headless: true (never rendered directly)
│       └── <author-slug>/<book>.md
├── static/                   # copied verbatim to the output (favicons, etc.)
├── themes/
│   └── kana-hugo-theme/      # git submodule (do NOT edit in place — see §11)
└── public/                   # generated site (git-ignored, created on build)
```

**Two folders that matter most:** everything you *write* lives in `content/`; everything you *tune*
lives in `hugo.toml`. Images that Hugo should resize go in `assets/`; files that should be copied
as-is go in `static/`.

---

## 4. Everyday workflow

Run these from the project root.

```bash
# Live dev server with drafts + future-dated posts, opens in browser
hugo server -D -F

# Dev server without drafts
hugo server

# One-off production build into ./public
hugo --gc --minify

# Build including drafts (e.g. to preview)
hugo -D
```

- Dev server: http://localhost:1313
- Drafts are posts/notes with `draft: true`. The homepage lists them (labelled `[DRAFT]`) while
  running in development mode.
- `-F` (`--buildFuture`) includes posts dated in the future.

> **Note:** `public/` and `resources/_gen/` are build artifacts and are git-ignored. Never edit them
> by hand; they are regenerated on every build.

---

## 5. Configuration reference (`hugo.toml`)

Open `hugo.toml` — it is commented throughout. The important entries:

### Site basics

| Key | What it does |
|---|---|
| `title` | Site name, shown in the header and page titles. |
| `baseURL` | Canonical URL of the *deployed* site. Set this before publishing. |
| `relativeURLs` / `canonifyURLs` | Keep links working regardless of where the site is hosted. Leave `true` unless you have a reason. |
| `locale` | Language/locale (e.g. `en-us`). |
| `theme` | Must be `"kana-hugo-theme"` (matches the folder name in `themes/`). |
| `enableRobotsTXT` | Generates `robots.txt`. |
| `disableKinds = ["taxonomy"]` | Hides the bare `/tags/` and `/categories/` list pages while keeping individual term pages like `/tags/hugo/`. Remove this if you *want* the list pages. |

### `[params]` — theme settings

| Param | Type | Purpose |
|---|---|---|
| `pagerSize` | int | Items per page (also the library's initial batch size). |
| `mainSections` | list | Sections shown on the homepage + main RSS. This project uses `["posts"]`. |
| `author` | string | Used in RSS `dc:creator` and JSON-LD. |
| `description` | string | Site description for meta tags and feeds. |
| `subtitle` | string | Appended to the homepage `<title>`. |
| `OpenGraph` / `TwitterCards` / `jsonld` | bool | Emit social/SEO meta tags. |
| `FediverseCreator` | string | `fediverse:creator` meta tag, e.g. `@you@instance.social`. |
| `MobileWebAppTitle` | string | iOS "add to home screen" title. |
| `imageSizes` | list | Widths (px) generated for responsive `srcset`, e.g. `[850, 710, 300]`. |
| `imageSizeString` | string | The HTML `sizes` attribute. Must list **smallest breakpoint first**, unconditional default last. |
| `headerIcon` | string | Path to an image in `assets/`, shown next to the site title. Remove the line to hide the icon. |
| `policies` | string | Path to a policies page linked in the footer. |
| `license` | table | `name`, `title`, `link` shown in the footer copyright line. |

### Outputs — required for search

```toml
[outputs]
home = ["HTML", "RSS", "JSON"]
```

The `JSON` output on the home page builds the search index (`/index.json`). **Do not remove it** or
search stops working.

### Permalinks

```toml
[permalinks]
posts = "/:year/:month/:slug/"
```

Post URLs become `/2026/01/welcome-to-the-blog/`. The `:slug` is taken from the post **title**
(lower-cased, hyphenated). Override per post with a `slug:` front-matter field.

### Markdown / highlighting

```toml
[markup.highlight]
noClasses = false          # use CSS classes (the theme styles .chroma)

[markup.goldmark.renderer]
unsafe = true              # allows raw HTML in Markdown (used by the homepage hero)

[markup.goldmark.parser.attribute]
block = true               # enables {file="..."} attributes on code fences
```

---

## 6. Menus

Menus are defined in `hugo.toml` under `[[menus.main]]` (header) and `[[menus.footer]]` (footer).

```toml
[[menus.main]]
name = "About"          # label shown
title = "About me"      # tooltip
pageRef = "/about"      # link to an existing page (by path, without trailing slash)
weight = 10             # lower = further left

[[menus.footer]]
name = "RSS"
url = "/index.xml"      # use `url` for external or special links
weight = 50
```

Use `pageRef` to point at a page and `url` for anything else (external links, `mailto:`, feeds).
Delete entries to remove them; add entries to extend the menu.

---

## 7. Content: the four things you can write

All content is Markdown with YAML front matter (`---` … `---` at the top).

### 7a. Posts (section: `posts`)

Long-form blog posts. Put them in `content/posts/<year>/`.

```yaml
---
title: "My New Post"
date: 2026-04-01T09:00:00-05:00
description: "One-sentence summary used for SEO/feeds."
draft: false
images:
  - /images/dev-image-01.jpeg      # first entry = featured image (must exist in assets/)
categories:
  - Writing
tags:
  - hugo
  - blogging
---
```

Create one with the archetype (fills in a title/date for you):

```bash
hugo new content posts/2026/04-01-my-new-post.md
```

Or just create the file by hand — the archetype is a convenience, not a requirement.

- The featured image is the **first** entry in `images:`. It is resized to WebP `srcset` sizes
  automatically. The file must live under `assets/` (here, `assets/images/…`).
- `categories` and `tags` generate term pages (e.g. `/tags/hugo/`).
- The `/blog` page (built from `content/blog.md`, `type: blog`) lists **all** posts.
- Posts appear on the homepage (latest 10) and in `/index.xml`.

### 7b. Notes (section: `notes`)

Short, tweet-like entries rendered **inline** on `/notes` — each note does *not* get its own page
(this is controlled by the `cascade` block in `content/notes/_index.md`).

```yaml
---
title: "Monday, 5 January, 2026"
date: 2026-01-05T00:00:00-06:00
---
```

Notes are grouped by year and have their **own RSS feed** at `/notes/index.xml`, separate from the
main feed. Create one:

```bash
hugo new content notes/2026/01-05-a-short-note.md
```

### 7c. Library books (section: `books` — headless)

The reading library at `/library` reads a **headless** bundle: `content/books/index.md`
(`headless: true`) whose child pages are the books. Each book lives in a folder named after the
author slug:

```
content/books/index.md                 # headless wrapper — leave as is
content/books/newport-cal/deep-work.md # one file per book
```

```yaml
---
title: "Deep Work"
author: "Cal Newport"
rating: 5                 # 0–5 stars
finished:
  - "2025-11-02"          # list every read date (re-reads supported)
  - "2023-03-19"
links:
  amazon: "https://..."
  openlibrary: "https://..."
  goodreads: "https://..."
---
```

The `/library` page automatically adds:

- filter-by-year, filter-by-author, and filter-by-rating controls (clickable summary cards),
- sorting by title / author / rating / finished date,
- a statistics panel (books per year, top authors, rating distribution),
- progressive loading in batches of `pagerSize`.

`content/library.md` (`type: library`, `layout: library`) activates the page. Create a book with:

```bash
hugo new content books/<author-slug>/my-book.md
```

### 7d. Standalone pages

About, Uses, Now, and Policies are plain pages (`type: page`). Search (`content/search.md`) and the
blog/library pages use `layout:` to pick a template. Add a page with:

```bash
hugo new content my-page.md
```

---

## 8. Writing: Markdown, shortcodes, code blocks, images

### Code blocks (Gruvbox highlighting)

Fenced blocks are highlighted automatically. Add attributes for extras:

````markdown
```go {file="main.go"}
package main
```
````

- `{file="name.ext"}` — shows a filename label above the block.
- `{details=true}` — wraps the block in a collapsible `<details>`. Add `{open=true}` to expand it by
  default. Optional `{summary="Click to expand"}` sets the summary text.

### Images

Two ways to add images:

1. **Markdown image** (file must be in `assets/`):
   ```markdown
   ![alt text](/images/photo.jpg "Optional caption")
   ```
   Rendered responsively (WebP `srcset`, lazy-loaded). A title becomes a `<figcaption>`.

2. **`asset-image` shortcode** — explicit size control:
   ```
   {{< asset-image "images/photo.jpg" "800" "600" "Alt text" >}}
   ```
   Positional args: path (relative to `assets/`), width, height, alt.

### Raw HTML

Because `markup.goldmark.renderer.unsafe = true`, you can drop HTML directly into Markdown. The
homepage (`content/_index.md`) uses this for its hero layout.

---

## 9. Customizing the look

Styles are SCSS compiled by Hugo from the **theme**'s `assets/scss/`. To override without touching
the submodule, create matching files under the site's own `assets/` — Hugo prefers your files over
the theme's.

The palette lives in the theme's `assets/scss/components/variables.scss`:

```scss
$primary:         #c8804a;  // warm terracotta (headings, accents)
$background:      #1a1816;  // warm near-black
$background-code: #282828;  // code block background
$link:            #d4a87a;  // warm amber
$text:            #cec6bc;  // body text
$text-bright:     #ede6dd;  // bright text
$font-family: -apple-system, BlinkMacSystemFont, "Helvetica Neue", Arial, sans-serif;
```

**To change colors/fonts**, copy the file into your project and edit it:

```bash
mkdir -p assets/scss/components
cp themes/kana-hugo-theme/assets/scss/components/variables.scss assets/scss/components/variables.scss
# now edit assets/scss/components/variables.scss — your copy wins
```

Since `main.scss` lives in the theme and `@import`s `components/variables.scss`, your copy of the
variables file is picked up automatically. For deeper changes, also override whole files (e.g.
`assets/scss/components/global.scss`) the same way, or override `assets/scss/main.scss` entirely.

Syntax highlighting colors are in the theme's `assets/scss/gruvbox.scss` — override it the same way
if you want a different palette.

> Gestalt: to restyle **anything**, mirror its path under your own `assets/` (or `layouts/`) and
> edit your copy. Never edit files inside `themes/`.

---

## 10. Overriding theme templates

Hugo resolves layouts with a lookup order that prefers your site's `layouts/` over the theme's. To
change a template, copy the theme file to the same relative path under `layouts/` and edit it.

```bash
# Example: change the header
mkdir -p layouts/_partials
cp themes/kana-hugo-theme/layouts/_partials/header.html layouts/_partials/header.html
```

Key templates in the theme:

| File | Role |
|---|---|
| `layouts/baseof.html` | Page skeleton (head/header/main/footer). |
| `layouts/home.html` | Homepage hero + recent posts. |
| `layouts/single.html` | Individual post/page. |
| `layouts/blog/blog.html` | `/blog` archive. |
| `layouts/library/library.html` | `/library` reading page. |
| `layouts/notes/list.html` | `/notes` archive. |
| `layouts/search.html` | `/search`. |
| `layouts/_partials/head.html` | `<head>`: styles, meta, feeds, SEO. |
| `layouts/_partials/header.html` | Site title/icon + main menu. |
| `layouts/_partials/footer.html` | Footer menu + copyright. |
| `layouts/_markup/render-codeblock.html` | Code-block rendering. |
| `layouts/_markup/render-image.html` | Markdown image rendering. |
| `layouts/shortcodes/asset-image.html` | The `asset-image` shortcode. |

Front matter can also select a template directly with `layout:` (as `library.md` and `blog.md` do).

---

## 11. Updating the theme

The theme is a git submodule, so it stays isolated from your content and is easy to update.

```bash
# Update to the latest commit on the theme's default branch
git submodule update --remote themes/kana-hugo-theme

# Then rebuild
hugo --gc --minify
```

If you ever clone this project elsewhere, fetch the submodule contents first:

```bash
git clone --recurse-submodules <your-repo-url>
# or, in an existing clone:
git submodule update --init --recursive
```

**Do not edit files inside `themes/kana-hugo-theme/`** — your changes would be overwritten on the
next update. Always override into `layouts/` or `assets/` instead (§9–§10).

---

## 12. Search

Search is fully client-side — no server or third-party service.

- The index is the home page's JSON output (`/index.json`), built by `layouts/index.json`.
- It indexes standalone pages plus posts (title, tags, categories, and body text).
- `/search` loads `scripts/search.js` and reads the query from `?s=`.
- To include a new section in search, adjust `layouts/index.json` (override it in your `layouts/`).

Keep `home = ["HTML", "RSS", "JSON"]` in `hugo.toml`, or search breaks.

---

## 13. SEO, feeds, and social

Handled automatically when the relevant `[params]` flags are on:

- Open Graph + Twitter Card tags, canonical URLs, and JSON-LD (`jsonld = true`).
- `robots.txt` (`enableRobotsTXT = true`) and a custom `sitemap.xml`.
- Site RSS at `/index.xml` and a notes-only feed at `/notes/index.xml`.
- Federation meta (`FediverseCreator`) for Mastodon-style verification.

Set `baseURL` to your real domain before deploying so canonical/absolute URLs are correct.

---

## 14. Deploying

1. Set `baseURL` in `hugo.toml` to your production URL.
2. Build: `hugo --gc --minify` (output in `public/`).
3. Upload/serve `public/`.

Common hosts (all have first-class Hugo support):

- **Netlify / Cloudflare Pages** — build command `hugo --gc --minify`, publish directory `public`.
  Set `HUGO_VERSION` to `0.167.0` (or newer) in the build environment. If the host clones the repo
  itself, make sure submodules are fetched (`git submodule update --init --recursive`).
- **GitHub Pages** — add a GitHub Actions workflow that checks out with submodules, installs Hugo
  extended, and publishes `public/`.
- Any static host or even `file://` — thanks to `relativeURLs`, the built files work from
  subdirectories too.

Remember to commit the submodule pointer (`themes/kana-hugo-theme`) so hosts know which theme
revision to fetch.

---

## 15. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| `TOCSS: failed to transform ... this feature is not available in your current Hugo version` | You're using non-extended Hugo. Install the **extended** build. |
| Pages build but CSS/JS 404s | You opened a page without a trailing slash, or `baseURL` is wrong. Use the dev server or a proper static host. |
| Search returns nothing | `home` output is missing `"JSON"` in `hugo.toml`, or `/index.json` wasn't built. |
| Header icon missing / build error about `HeaderIcon` | The `headerIcon` file must exist under `assets/`. Remove the `headerIcon` line to hide it. |
| Featured image not showing | The first `images:` entry must exist under `assets/` and be referenced with a leading `/` (e.g. `/images/foo.jpeg`). |
| Library page empty | Books must be under `content/books/` with a headless `index.md`; the section is `books`. |
| A note gets its own page | Individual notes are disabled via `cascade` in `content/notes/_index.md`; keep that block. |
| Draft not visible | Run with `-D` (`hugo server -D`). |
| Theme files show as empty after a fresh clone | Run `git submodule update --init --recursive`. |

---

## 16. Quick-reference: common tasks

```bash
# New blog post
hugo new content posts/2026/04-01-my-post.md

# New note
hugo new content notes/2026/04-01-a-note.md

# New library book
hugo new content books/<author-slug>/<book-slug>.md

# New standalone page
hugo new content my-page.md

# Preview everything (drafts + future)
hugo server -D -F

# Production build
hugo --gc --minify

# Update the theme
git submodule update --remote themes/kana-hugo-theme
```

---

## 17. Learn more

- Hugo docs: https://gohugo.io/documentation/
- Kana theme: https://github.com/ChrisWiegman/kana-hugo-theme
- Theme demo: https://chriswiegman.com
- Hugo themes gallery: https://themes.gohugo.io/
