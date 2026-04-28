# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
yarn install
yarn dev      # http://localhost:4321
yarn build    # static output → dist/
yarn preview  # serve dist/
yarn check    # astro check (TypeScript + content schema)
yarn format   # prettier
```

No test suite. `yarn check` is the closest thing to a test gate; run it before claiming a build-breaking schema change works.

Node 20+. This repo uses **yarn** (`packageManager: yarn@1.22.22`, lockfile `yarn.lock`). `package-lock.json` is in `.gitignore` — running `npm install` locally won't pollute the repo, but the source of truth for dependency versions is `yarn.lock`. Cloudflare Pages also installs via yarn (auto-detected from lockfile).

## Architecture overview

This is a customized fork of the [Bookworm Light Astro](https://github.com/themefisher/bookworm-light-astro) template (Astro 6, MDX, React, Tailwind v4). Three layers worth knowing before editing anything:

### 1. Content pipeline

Content lives under `src/content/` as Markdown/MDX. Schemas are defined in `src/content.config.ts` for five collections: `posts`, `pages`, `about`, `contact`, `authors`. **The Zod schema there — not `.pages.yml` — is the source of truth for required frontmatter.**

`src/lib/contentParser.astro` exports `getSinglePage(collection)` used by every page that lists or renders entries. It does two filtering passes invisibly:

- Strips entries whose `id` starts with `-` (these are config-style entries like `-index.md`)
- Strips entries with `draft: true`

When debugging "why doesn't my post show up", check both filters before checking routing.

### 2. Routing

Files in `src/pages/`:

- `/` (`index.astro`) and `/page/{n}` — paginated post listings
- `/blog/` (`blog/index.astro`) — full archive
- `/blog/{post-id}` (`blog/[single].astro`) — single post; `post-id` is the file's content-collection id (filename without extension)
- `/tags/`, `/categories/`, `/authors/` — taxonomy indices, all driven by post frontmatter
- `/[regular].astro` — catch-all for top-level pages from the `pages` collection (e.g. `/elements`, `/privacy-policy`)

Post links throughout the theme hardcode `/blog/${post.id}`. Don't move posts to a different route prefix without updating `Posts.astro`, `SimilarPosts.astro`, `PostSingle.astro`, and `Share.astro`.

### 3. Site config

Site-wide settings split across four JSON files in `src/config/`:

- `config.json` — site URL, pagination, blog folder name, metadata
- `menu.json` — header & footer nav links (manually maintained; stale links 404 silently)
- `social.json` — footer social icons (uses `FaXxx` icon name strings)
- `theme.json` — fonts (parsed by `astro.config.mjs` into Astro Fonts) and colors

Changing a config value usually has higher impact than editing components — check these first when something looks "themed".

## Pages CMS + Cloudflare Pages workflow

`.pages.yml` configures [Pages CMS](https://pagescms.org) (used at [app.pagescms.org](https://app.pagescms.org)) for browser/mobile editing. Key conventions:

- Posts filename pattern: `{year}-{month}-{day}-{fields.slug}.md` — `slug` is a required field (`^[a-z0-9-]+$`); empty slugs produce broken filenames like `2026-04-28-.md`
- Media: CMS uploads land in `public/images/`; frontmatter references them as `/images/foo.jpg` (NOT `/src/assets/...` — that path won't resolve at runtime)
- `categories` and `authors` are `hidden: true` in the CMS but defaulted by the schema, so CMS users don't see them but they still populate

Cloudflare Pages auto-builds on push to `main` and deploys to `beancount-notes.chialin.me`. CMS edits commit directly to `main` (commit author shows "via Pages CMS"), so a push from local + a save from the CMS can race — `git pull` before editing locally if the CMS has been used recently.

## Image handling

The theme uses `<Image>` from `astro:assets`. With paths like `/images/foo.jpg` (public folder), Astro skips optimization and emits a plain `<img>` — this is the intended behavior here. To get optimized output, an image must be imported from `src/assets/` and the schema field switched from `z.string()` to `image()`. Don't do that without also reworking `.pages.yml` media config.

## Reference docs

- `docs/superpowers/specs/2026-04-28-beancount-blog-design.md` — design spec
- `docs/superpowers/plans/2026-04-28-beancount-blog.md` — implementation plan with task-by-task history (including post-deploy fixes)
- `docs/notes/2026-04-28-cloudflare-pages-custom-domain.md` — runbook for the Cloudflare custom-domain setup (external DNS path)
