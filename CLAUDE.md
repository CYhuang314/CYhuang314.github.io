# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A Hexo 8 static blog ("CYhuang's Blog", `zh-TW`, `Asia/Taipei`). Everything is stock Hexo — there is no application code, no test suite, and no linter. Work here is content authoring plus configuration.

## Commands

```bash
npm run server            # dev server with live reload at http://localhost:4000
npm run build             # hexo generate → static site into public/
npm run clean             # delete public/ and db.json (run after config/theme changes)
npm run deploy            # hexo deploy (see Deployment below — not yet configured)

npx hexo new "Post Title"        # new post from scaffolds/post.md → source/_posts/
npx hexo new draft "Title"       # → source/_drafts/ (not rendered unless render_drafts: true)
npx hexo new page "about"        # → source/about/index.md
npx hexo publish "draft-name"    # move a draft into source/_posts/
npx hexo server --draft          # preview drafts
```

`hexo clean` matters more than usual: Hexo caches rendered output in `db.json`, and stale cache is the usual cause of "my change didn't show up". Prefer `npm run clean && npm run build` when config, scaffolds, or theme files change.

## Structure

- `source/_posts/` — published posts, one Markdown file each with YAML front matter (`title`, `date`, `tags`, `categories`).
- `scaffolds/` — templates `hexo new` copies for `post`, `draft`, and `page`. Edit these to change the default front matter of new content.
- `_config.yml` — site config. Permalinks are `:year/:month/:day/:title/`; the `title` slug comes from the post's `title` front matter, so renaming a title changes its URL.
- `_config.landscape.yml` — theme config override for the `landscape` theme (currently empty). Hexo merges `_config.[theme].yml` over the theme's own config; put theme customization here rather than editing files under `node_modules/hexo-theme-landscape/`.
- `themes/` — empty. The active theme is installed as the npm dependency `hexo-theme-landscape`. To fork/customize heavily, copy it into `themes/landscape/` so it stops being overwritten by npm.
- `public/`, `db.json` — build artifacts, gitignored, never edit by hand.

## Rendering

Markdown via `hexo-renderer-marked`, templates via EJS, stylesheets via Stylus. Syntax highlighting is `highlight.js` with line numbers on. Index, archive, category, and tag pages are produced by the corresponding `hexo-generator-*` plugins; adding a plugin to `package.json` is what creates new route types.

## Deployment

`_config.yml` sets `url: https://CYhuang314.github.io` but `deploy.type` is empty and no deployer package (e.g. `hexo-deployer-git`) is installed, so `npm run deploy` is currently a no-op. Wiring up deployment means installing a deployer and filling in the `deploy:` block.
