# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A Hexo 8 static blog ("CYhuang's Blog", `zh-TW`, `Asia/Taipei`) using the Fluid theme, auto-deployed to GitHub Pages at https://cyhuang314.github.io/. There is no application code, no test suite, and no linter — work here is content authoring plus configuration.

`README.md` is the user-facing guide (written in Traditional Chinese) covering the writing workflow and troubleshooting. Keep it in sync when changing the build or deploy setup.

## Node version

Hexo 8.1.2 requires **Node >= 20.19.0**. The system Node is 18.19.1, so `nvm use` (reads `.nvmrc` → 22) is required before running any hexo command. In non-login shells nvm must be sourced first:

```bash
export NVM_DIR="$HOME/.nvm"; . "$NVM_DIR/nvm.sh"; nvm use
```

## Commands

```bash
npm run server            # dev server with live reload at http://localhost:4000
npm run build             # hexo generate → static site into public/
npm run clean             # delete public/ and db.json

npx hexo new "Post Title"        # new post from scaffolds/post.md → source/_posts/
npx hexo new draft "Title"       # → source/_drafts/ (not rendered unless render_drafts: true)
npx hexo new page "about"        # → source/about/index.md
npx hexo publish "draft-name"    # move a draft into source/_posts/
npx hexo server --draft          # preview drafts
```

`hexo clean` matters more than usual: Hexo caches rendered output in `db.json`, and stale cache is the usual cause of "my change didn't show up". Always `npx hexo clean && npx hexo generate` after touching `_config.yml`, `_config.fluid.yml`, or `scaffolds/`. The dev server picks up Markdown edits live but **not** config edits — those need a restart.

## Deployment

Push to `main` → `.github/workflows/deploy.yml` runs `npm ci`, `hexo clean`, `hexo generate`, and uploads `public/` via `actions/upload-pages-artifact` → `actions/deploy-pages`.

- `npm run deploy` / the `deploy:` block in `_config.yml` are **unused** — there is no deployer package installed. Do not suggest `hexo deploy`.
- GitHub Pages `build_type` must stay `workflow`. If it reverts to `legacy`, Pages serves raw Markdown from `main` instead of the built site:
  ```bash
  gh api repos/CYhuang314/CYhuang314.github.io/pages --jq '.build_type'
  gh api -X PUT repos/CYhuang314/CYhuang314.github.io/pages -f build_type=workflow
  ```
- The workflow uses `npm ci`, so `package-lock.json` must be committed in sync with `package.json` or the build fails.
- The theme is the npm dependency `hexo-theme-fluid` (v1.9.9), deliberately **not** a git submodule, so CI needs no submodule handling. Keep it that way.

## Structure

- `source/_posts/` — published posts, one Markdown file each with YAML front matter.
- `source/about|tags|categories/index.md` — pages the Fluid navbar links to; each needs the matching `layout:` (`about` / `tags` / `categories`) or it 404s.
- `scaffolds/` — templates `hexo new` copies. Edit these to change default front matter.
- `_config.yml` — site config. Permalinks are `:year/:month/:day/:title/`, so the URL derives from the post's `title` front matter — renaming a title changes its URL. Chinese titles get URL-encoded; use a per-post `permalink:` to override.
- `_config.fluid.yml` — full copy of the theme's default config (Fluid's documented approach). **All theme customization goes here**; never edit `node_modules/hexo-theme-fluid/`. Hexo deep-merges this over the theme's own `_config.yml`.
- `themes/` — empty and unused; the theme comes from npm.
- `public/`, `db.json` — build artifacts, gitignored, never edit by hand.

## Fluid specifics

- Fluid generates its own search index (`scripts/generators/local-search.js` → `/local-search.xml`). Do **not** install `hexo-generator-search` — it only adds a redundant `search.xml`.
- Fluid-specific front matter: `index_img` (home list thumbnail), `banner_img` (post header image), `sticky` (higher number pins higher).

## Rendering

Markdown via `hexo-renderer-marked`, templates via EJS, stylesheets via Stylus. Syntax highlighting is `highlight.js` with line numbers on. Index, archive, category, and tag routes come from the `hexo-generator-*` plugins in `package.json`.
