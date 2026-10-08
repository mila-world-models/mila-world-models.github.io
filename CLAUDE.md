# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Hugo static site for the Mila World Models Reading Group. There is no theme, no JS build, and no tests. Everything is hand-written templates in `layouts/`, one stylesheet in `assets/css/site.css`, and YAML data files. Pushing to `main` deploys to GitHub Pages via `.github/workflows/hugo.yml`.

## Commands

- `hugo server`: local preview. `baseURL = "/"` in `hugo.toml` on purpose, so this works without flags.
- `hugo --minify`: production build into `public/`. CI also passes `--baseURL` with the real Pages URL.

CI uses **Hugo extended 0.163.3**. Use a recent extended build locally: templates use `hugo.Data` (not `site.Data`), and image processing (`.Fill`, `.Fit`, webp) needs the extended edition. Hugo is not installed on this machine by default (`brew install hugo`).

## How the site is put together

- **Homepage** (`layouts/index.html`) is a single long page. It renders the intro from `content/_index.md`, the "When/Where/Updates" facts from `[params]` in `hugo.toml`, then the schedule, organizers, sponsors, and contact sections from `data/sessions.yaml`, `data/organizers.yaml`, and `data/sponsors.yaml`. A section disappears when its data file is empty, and each optional field is skipped when blank.
- **Other pages** (`content/*.md`) all use `layouts/_default/single.html`. Front-matter flags turn on extras: `form: true` adds the contact form partial, and `gallery: true` adds the photo gallery partial.
- **Most content edits are data edits, not template edits.** Each YAML file starts with a comment block documenting its schema. Read it before adding entries.
- **Image lookups use filename conventions.** Templates find images with `resources.GetMatch` under `assets/images/`:
  - Organizers: `organizers/<lowercased first name>.*`, or the `photo:` override. Cropped to a square at build time.
  - Sponsors: `sponsors/<lowercased name, spaces removed>.*`, or the `logo:` override.
  - Gallery: `gallery/YYYY-MM-DD-<anything>-NN.ext`. The first 10 characters of the filename are the session date used for grouping, and `data/gallery.yaml` maps those dates to labels. HEIC and other non-processable files are silently skipped (`partials/gallery-photos.html`). The "Photos" nav link only appears when at least one photo exists.
  - Images in `assets/` are processed by Hugo. `static/` holds only files served as-is (favicon, `images/og.png` link-preview image).
- **Session `status`** (`"Upcoming"`, `"Past"`, ...) becomes both a badge and a CSS class via `urlize` (`is-upcoming`, `badge--past`). The legacy `upcoming: true` still maps to "Upcoming". Sessions are listed in file order, newest first. Nothing sorts them automatically.
- **Contact routing.** `partials/contact-url.html` points every "Contact" link at `params.contactURL` if set, or `/contact/` otherwise. The form posts to FormSubmit (`params.formEndpoint`), which forwards to the private organizers' group and redirects to `/thanks/`. Keep this separate from the public mailing list (`params.mailingListURL` / `mailingListSubscribe`): anything sent to the list goes to every subscriber.
- **Raw HTML is allowed.** Goldmark `unsafe = true` is set, and several fields are piped through `safeHTML` (`paper`, `presenter`, `note`, sponsor `note`, `params.where`) or `markdownify` (`abstract`, `bio`).
