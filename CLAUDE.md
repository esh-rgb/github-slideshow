# CLAUDE.md — AI Assistant Guide for github-slideshow

## Project Overview

**github-slideshow** is a Jekyll-based slideshow application originally created as a GitHub Learning Lab course template. It converts Markdown files (`_posts/`) into interactive web-based presentations using [reveal.js](https://github.com/hakimel/reveal.js/).

The site is hosted via **GitHub Pages** and built automatically on push to the main branch.

---

## Technology Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Static Site Generator | Jekyll | 3.9.0 |
| Presentation Framework | reveal.js | 3.9.2 (npm) |
| Hosting | GitHub Pages | via `github-pages` gem |
| Markdown Processor | kramdown | bundled with Jekyll |
| Syntax Highlighting | Rouge | 3.19.0 |
| HTML Validation | html-proofer | >= 3.13.0 |
| Backend Language | Ruby | (via Gemfile) |
| Frontend Language | HTML/CSS/JavaScript | — |

---

## Repository Structure

```
github-slideshow/
├── CLAUDE.md                  # This file
├── README.md                  # Project introduction for GitHub Learning Lab users
├── _config.yml                # Jekyll + reveal.js configuration
├── index.html                 # Entry point: iterates _posts/ to render slides
├── Gemfile / Gemfile.lock     # Ruby gem dependencies
├── package-lock.json          # npm lock (reveal.js only)
├── .editorconfig              # Code style rules
├── .gitignore                 # Ignores _site/, .bundle, etc.
├── _posts/                    # Slide content as Markdown files
│   └── 0000-01-01-intro.md   # Example/default intro slide
├── _layouts/
│   ├── presentation.html      # Full reveal.js slideshow wrapper
│   ├── slide.html             # Single slide view layout
│   └── print.html             # Print/PDF export layout
├── _includes/
│   ├── head.html              # HTML <head> with CSS imports
│   ├── script.html            # reveal.js JS initialization
│   └── slide.html             # Per-slide <section> template
└── script/
    ├── setup                  # Install Ruby/gem dependencies
    ├── server                 # Start local dev server
    ├── cibuild                # Build + validate HTML (CI)
    └── stage                  # Deploy to staging (GitHub Enterprise)
```

---

## Development Workflow

### Initial Setup

```sh
script/setup
```

This script:
- Installs Ruby via rbenv (if `.ruby-version` is present)
- Installs Bundler and gem dependencies via `bundle install`
- Initializes git submodules

### Local Development Server

```sh
script/server
# Equivalent to: bundle exec jekyll serve
# Visit: http://localhost:4000
```

Pass additional Jekyll flags through the script (e.g., `script/server --livereload`).

### Build & Validate (CI)

```sh
script/cibuild
# Equivalent to:
#   bundle exec jekyll build --baseurl "."
#   htmlproofer _site/index.html --empty-alt-ignore
```

This is the canonical CI check. Always ensure it passes before merging.

### Staging Deployment

```sh
script/stage
# Builds site with a staging baseurl and pushes to GitHub Enterprise Pages
```

---

## Adding Slides

Slides are Jekyll posts stored in `_posts/`. Each file must:

1. Follow the naming convention: `YYYY-MM-DD-slug.md`
2. Include front matter with `layout: slide` and optionally a `title`

**Example slide file** (`_posts/0000-01-01-my-slide.md`):

```markdown
---
layout: slide
title: "My Slide Title"
---

Content goes here. Supports standard Markdown.
```

**Slide ordering**: Jekyll renders posts in date order. Since all slides typically use `0000-01-01`, use the slug portion to distinguish slides. `index.html` iterates posts in **reversed** order via `{% for post in site.posts reversed %}`.

### Slide Front Matter Options

| Key | Description |
|-----|-------------|
| `layout` | Must be `slide` |
| `title` | Displayed as an `<h1>` on the slide |
| `slide-id` | Sets the `id` attribute on the `<section>` element |
| `classes` | Array of CSS classes added to the `<section>` element |
| `data` | Key-value pairs rendered as `data-*` attributes for reveal.js |

---

## Configuration

All site and reveal.js settings live in `_config.yml`.

### Key Jekyll Settings

```yaml
timezone: Europe/Berlin
markdown: kramdown
highlighter: rouge
permalink: "/:title"
plugins:
  - jemoji
```

### Reveal.js Settings (under `reveal:` key)

Common settings to adjust:

```yaml
reveal:
  transition: linear         # Slide transition style
  backgroundTransition: slide
  controls: false            # Hide navigation arrows
  progress: true             # Show progress bar
  history: true              # Browser history integration
  center: true               # Vertically center slides
  width: 1000
  height: 920
```

### Theme

The solarized theme variant is set via:

```yaml
solarized:
  theme: dark    # or: light
```

The reveal.js CSS theme is set in `_includes/head.html` (currently `moon`).

---

## Dependencies

### Ruby (Gemfile)

- `github-pages` (>= 207) — Jekyll + GitHub Pages-compatible plugin set
- `html-proofer` (>= 3.13.0) — HTML validation in CI
- `tzinfo-data` — Windows timezone data support

Install/update with:
```sh
bundle install
bundle update
```

### JavaScript (npm)

- `reveal.js@3.9.2` — loaded from `node_modules/`

The `node_modules/` directory is **not** in `.gitignore` and is committed directly.
To reinstall: `npm install`

---

## Code Style & Conventions

From `.editorconfig`:

- **Default**: tabs for indentation, 4-space tab width, LF line endings, UTF-8
- **JSON, JS, CSS, SCSS, YAML, HTML**: spaces, 2-space indent
- **Markdown**: spaces, 4-space indent, no trailing whitespace trimming, insert final newline

Always respect these rules when editing any file in the repo.

---

## Testing & Quality Checks

There are **no unit or integration tests**. Quality is enforced via:

1. **html-proofer** — validates the built HTML output (`_site/index.html`)
   - Run via `script/cibuild`
   - Ignores empty alt attributes (`--empty-alt-ignore`)

2. **Manual review** — view slides locally with `script/server`

Before committing, ensure `script/cibuild` runs without errors.

---

## Deployment

- **Production**: GitHub Pages builds automatically from commits to the default branch
- **No CI pipeline configured** (CircleCI was removed; no GitHub Actions present)
- The `_site/` directory is `.gitignore`d — GitHub Pages builds remotely

---

## Important Notes for AI Assistants

- **Do not modify `node_modules/`** — reveal.js is committed directly; only update via `npm install`
- **Slide files drive all content** — the entire presentation is generated from `_posts/*.md`
- **`_config.yml` controls presentation behavior** — reveal.js options are passed through Jekyll templating in `_includes/script.html`
- **No test suite** — validate changes by running `script/cibuild` and inspecting the output
- **GitHub Pages constraints** — only Jekyll plugins whitelisted by the `github-pages` gem are supported; do not add arbitrary Jekyll plugins
- **`baseurl` is commented out** — leave it commented unless deploying to a subpath; local dev uses root `/`
