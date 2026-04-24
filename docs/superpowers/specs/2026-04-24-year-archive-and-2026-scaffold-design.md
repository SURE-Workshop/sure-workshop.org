# SURE Workshop: 2025 Archive + 2026 Scaffold

## Goal

Restructure the Jekyll site so the current SURE 2025 content is preserved as a stable archive at `/2025/*` and the root of the site (`/`, `/cfp`, `/program`, `/papers`, `/organizers`, `/faqs`) becomes the active SURE 2026 edition. 2026 pages are scaffolds with placeholders; they will not necessarily be deployed immediately.

## Non-Goals

- No visual redesign. Theme, layout, colors, banner, and favicon stay as they are.
- No data migration tooling. This is a one-time restructure.
- No backfill of pre-2025 editions (none exist — 2025 was the 1st edition).

## URL Structure

| URL                               | Edition | Source file                 |
| --------------------------------- | ------- | --------------------------- |
| `/`                               | 2026    | `index.md`                  |
| `/cfp/`                           | 2026    | `cfp.md`                    |
| `/program/`                       | 2026    | `program.md`                |
| `/papers/`                        | 2026    | `papers.md`                 |
| `/organizers/`                    | 2026    | `organizers.md`             |
| `/faqs/`                          | 2026    | `faqs.md`                   |
| `/2025/`                          | 2025    | `2025/index.md`             |
| `/2025/cfp/`                      | 2025    | `2025/cfp.md`               |
| `/2025/program/`                  | 2025    | `2025/program.md`           |
| `/2025/papers/`                   | 2025    | `2025/papers.md`            |
| `/2025/organizers/`               | 2025    | `2025/organizers.md`        |
| `/2025/faqs/`                     | 2025    | `2025/faqs.md`              |

Each 2025 page uses `permalink: /2025/<page>/` in front matter.

## Repo Layout

```
index.md, cfp.md, faqs.md, organizers.md, papers.md, program.md   ← 2026 (active)
2025/index.md, 2025/cfp.md, 2025/faqs.md, 2025/organizers.md,
     2025/papers.md, 2025/program.md                              ← 2025 (frozen)

_data/
  2025/
    accepted_papers.yml, cfp.yml, faqs.yml, index.yml,
    map.yml, organizers.yml, program.yml     ← frozen snapshot of current _data
  2026/
    accepted_papers.yml, cfp.yml, faqs.yml, index.yml,
    map.yml, organizers.yml, program.yml     ← scaffolded 2026 content
```

The flat `_data/*.yml` files are removed. All data references become year-scoped.

## Page Front Matter Convention

Every page (root and archived) carries a `year` field:

```yaml
---
layout: cfp
title: Call for Papers
year: 2026
navorder: 2
---
```

Archived 2025 pages set `year: 2025`, add `permalink: /2025/<page>/`, and **drop** `navorder` so they don't appear in the main nav.

## Layout Changes

Layouts currently read `site.data.cfp`, `site.data.program`, etc. These become:

```liquid
{% assign ydata = site.data[page.year] %}
{% assign cfp = ydata.cfp %}
```

Same pattern for every data access. The `page.year` field is the single source of truth for which edition the page belongs to.

## Header and Navigation

`_includes/header.html` changes:

1. **Brand title** — conditional on `page.year`:
   - If `page.year != site.active_year` (new config key, set to `2026`), display `SURE {{ page.year }}`.
   - Otherwise display `site.title` (`SURE 2026`).
2. **Nav links** — still built from `site.pages | sort: 'navorder'`. Since 2025 pages drop `navorder`, they never appear. 2026 pages keep their existing `navorder` values.
3. **New nav item**: a "Past Editions" link at `navorder: 99` pointing to `/2025/`. Suppressed whenever `page.year != site.active_year` (i.e., on any archived page, not just 2025).
4. **Archive banner**: when `page.year != site.active_year`, render a small notice at the top of the content area:
   > This is the archived page for SURE {{ page.year }}. Visit the [current edition](/) for SURE {{ site.active_year }}.

The banner lives in `_layouts/default.html` (or `_includes/header.html`) so every layout inherits it.

## _config.yml Changes

```yaml
title: SURE 2026
active_year: 2026
```

Everything else (description, theme, plugins, baseurl, permalink) unchanged.

## 2026 Content Scaffold

All 2026 pages are copies of their 2025 counterparts with the following mechanical changes:

- `2025` → `2026` in prose.
- `sure25.hotcrp.com` → `sure26.hotcrp.com`.
- CCS 2025 / Taiwan co-location references → `TBD`.
- "1st SURE Workshop" → "2nd SURE Workshop".
- Specific dates (July 7, August 13, August 19, October 13) → `TBD`.
- `_data/2026/accepted_papers.yml` → emptied (empty list).
- `_data/2026/program.yml` → emptied / placeholder structure preserved.
- `_data/2026/organizers.yml` → copied unchanged (editable later).
- `_data/2026/cfp.yml`, `index.yml`, `faqs.yml`, `map.yml` → copied with dates/venue replaced by TBD.

## Assets

`assets/img/*`, `assets/css/*`, `assets/icons/*` are shared across editions and untouched.

## Testing

- `bundle exec jekyll serve` (via existing `serve.sh`).
- Verify `/` renders 2026 content with "SURE 2026" brand and no archive banner.
- Verify `/2025/` renders 2025 index with "SURE 2025" brand and the archive banner.
- Verify all 2025 sub-pages (`/2025/cfp/`, `/2025/program/`, etc.) resolve and show their original content.
- Verify main nav on 2026 pages shows CFP/Program/Papers/Organizers/FAQs + Past Editions.
- Verify main nav on 2025 pages does NOT show Past Editions.
- Verify internal links in 2025 pages (`./cfp.md`, `./program.md`) resolve correctly under the `/2025/` permalink prefix.

## Risks

- **Relative-link breakage on 2025 pages.** The current pages use `./cfp.md`-style links that Jekyll resolves at the page's own URL. Moved pages with a `/2025/` permalink should still resolve correctly, but this needs verification during testing.
- **Theme expectations.** The remote theme (`piazzai/jekyll-nagymaros`) may expect specific data keys at `site.data.<key>`. If any layout outside our control reads them directly, those layouts must be overridden locally. All layouts used by this site are already overridden in `_layouts/`, so the risk is contained.

## Out of Scope / Future

- Deploying 2026 (will be done when content is finalized).
- A `_data/common.yml` for truly shared data (e.g., Discord link, Twitter handle) — current duplication is small enough that this isn't worth building yet.
- Automating future year rollovers (3rd edition in 2027 etc.) — one more cycle of manual rollover is fine; patterns can emerge then.
