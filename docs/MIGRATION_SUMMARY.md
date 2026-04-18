# Migration Summary

Retrospective of the migration from the pre-migration static HTML site to
the Jekyll-based template (fork of
[ngoductuanlhp/ngoductuanlhp.github.io](https://github.com/ngoductuanlhp/ngoductuanlhp.github.io)).

The pre-migration state is preserved on the
[`legacy-backup`](https://github.com/Wuao652/Wuao652.github.io/tree/legacy-backup)
branch. The forward-looking plan lives in
[`MIGRATION_PLAN.md`](./MIGRATION_PLAN.md); this document describes what
actually happened.

## Shape of the migration

- **5 PRs, landed in order.** Each PR was scoped small enough to review
  on its own; the live site remained the pre-migration page through
  PRs #1–#3 and only flipped over in PR #4.
- **`legacy-backup` is untouched.** A full revert back to the static
  site is a single branch checkout away.
- **CI (`.github/workflows/jekyll-build.yml`) gates every PR.** It runs
  `bundle exec jekyll build` on Ruby 3.3 / Ubuntu when a `Gemfile` is
  present, and skips gracefully before that.

## What each PR delivered

### PR #1 — Infrastructure (`d0574ae`)

Docs + CI scaffolding, no runtime changes.

- Added `docs/MIGRATION_PLAN.md` (this file's sibling).
- Added `.github/workflows/jekyll-build.yml` with a `Gemfile`
  presence check so the workflow could land before Jekyll itself.
- Added a one-paragraph migration notice to `README.md`.

### PR #2 — Jekyll template skeleton (`49657a3`)

Landed the Jekyll build system and template assets alongside the
legacy site. The template's homepage was staged as `index-new.html`
so the live `index.html` kept rendering the pre-migration page.

- `Gemfile`, `Gemfile.lock`, `_config.yml`, `.gitignore`.
- `_sass/_base.scss`, `css/index.scss`, `css/cv.scss`, `js/index.js`.
- Minified vendor assets actually referenced by the template:
  `css/bulma.min.css` (207 KB), `js/fontawesome.all.min.js` (1.2 MB).
- `index-new.html` — the template homepage, staged.

Surprises resolved in this PR:

- **Upstream template's default branch is `master`, not `main`.**
  First raw-URL fetches 404'd; switched once.
- **Template shipped vendor duplicates we could skip.** Non-minified
  `bulma.css`, `fontawesome.all.css`, and `bulma.css.map` totalled
  ~3 MB of git weight; dropped in favour of only the `.min` files.
  Academicons loads from a CDN; `fontawesome.all.min.css` was not
  used by the homepage and was deferred to PR #3 (where `cv.html`
  needs it).
- **`wdm` gem fails on Ruby 3.4.** Removed from the `Gemfile` — it's
  a Windows-only dev-watch optimisation, harmless to drop.
- **Ruby 3.4 removed `csv`, `base64`, `logger`, `bigdecimal` from the
  default stdlib.** Jekyll 4.2.2 still requires them; added each as
  an explicit `Gemfile` dependency. On Ruby 3.3 (CI) these are no-ops.
- **`Gemfile.lock` platform resolution.** Locked for both
  `x64-mingw-ucrt` (local Windows) and `x86_64-linux` (CI).

### PR #3 — Content migration (`8df7dfe`)

Migrated every content block from the pre-migration `index.html`
into the template's YAML schema, and introduced `cv.html`.

- `_data/authors.yml` — 7 authors (Wuao with `is_me: true` + 6
  collaborators).
- `_data/publications.yml` — 3 entries.
- `_data/experience.yml` — Honda Research Institute.
- `_data/news.yml` — 5 items; the oldest carries `archived: true`.
- `_data/education.yml`, `employment.yml`, `services.yml` — support
  for the CV page.
- `cv.html` — new second page.
- `css/fontawesome.all.min.css` — vendor asset required by
  `cv.html` (the JS-only version was sufficient for the homepage).
- `_config.yml`: filled `linkedin_url`, `google_scholar`,
  `twitter_username`, `position`; pasted the real bio.
- `index-new.html`: repointed the hero portrait, CV link, and
  favicon at the legacy `images/` and `data/` files.

In-flight design decisions made during PR #3:

- **Explicit `archived: true` flag for news**, replacing the
  template's `forloop.index > 5` threshold. Gives per-item control
  and survives future news additions.
- **Toggle labels rewritten** from "Show more / Show less" to
  "Click to see old news / Hide old news".
- **Publications got their own `<section id="publications">`** with
  a visible `<h2>Publications</h2>`. The upstream template had that
  heading commented out, which left the navbar's `#publications`
  anchor with no target.
- **Duplicate arxiv id preserved faithfully.** Both *iNatSounds MAE*
  and *Audio Geolocation* point at `2505.18726` in the legacy page;
  copied as-is rather than guess. Open follow-up.

### PR #4 — Switchover (`36776b3`)

The cut-over. After this PR merged, `https://wuao652.github.io/`
served the template instead of the legacy static page.

- Deleted legacy `index.html` and `stylesheet.css`.
- `index-new.html` → `index.html`.
- `_config.yml` excludes pruned (`stylesheet.css` removed; `data/`
  removed so `data/WuaoLiu_ModernCV.pdf` is published).

While the PR was open, we also polished the freshly-live page:

- **Restored the ClustrMaps visitor widget** from the legacy site,
  in its own `<section>` between the back-to-top button and the
  footer, with Bulma styling and the same `<hr>` divider the other
  sections use. Map id carried over verbatim.
- **Deleted the template's hidden ClustrMaps block** (had the
  upstream author's `LDoksBua…` map id and label
  "ngoductuanlhp.github.io's clustrmaps"). `hidden="hidden"` but
  still shipped to `_site/` and visible in View Source.
- **Hero heading:** added `(刘武傲)` after "Wuao Liu" via a
  `.name-chinese` span (smaller, lighter weight).
- **Hero portrait:** capped at `max-width: 250px`, centered.
- **News + Experience list indents** bumped to `2.5rem`.
- **Section `padding-bottom` dropped to `0.5rem`** to balance the
  whitespace around section headings.
- **Footer credit expanded** to "Template by Tuan Duc Ngo. Feel
  free to borrow this template." — name links to the upstream
  author's site, "borrow this template" still links to the repo.

### PR #5 — Personalization, partial (`0343cfc`)

The input-independent items from the final PR. Remaining items
wait on external information (see follow-ups below).

- **SEO metadata** on both `index.html` and `cv.html`: canonical
  link, `<meta name="author">`, Open Graph (`og:*`), Twitter Card
  (`twitter:*`). Link previews in Slack / iMessage / search now
  render the profile photo + description.
- **Callout refactor**: `<span style="color:red">` in the hero bio
  moved into a `<span class="callout">` + new `.callout` rule in
  `css/index.scss` (colour: `firebrick`, matching the existing
  publication-awards / publication-venue-emph accents).
- **Feed hygiene**: trailing slash removed from `site.url` so
  `feed.xml` no longer emits `wuao652.github.io//feed.xml`. Split
  `site.description` (short, plain text — feed subtitle +
  `<meta description>`) from a new `site.bio` (rich markdown —
  hero render). The feed subtitle no longer leaks literal `**` /
  `<span>` syntax to RSS readers.
- **Google Analytics blocks conditionalized** behind
  `{% if site.google_analytics %}` on both pages, so we no longer
  emit a broken `gtag/js?id=` script while the id is empty.
- **X profile link** added to the hero profile-links row.

## Things that surprised us

| # | Surprise | How we resolved it |
|---|----------|--------------------|
| 1 | Upstream default branch is `master` | Switched the raw-URL fetches |
| 2 | Ruby 3.4 dropped `csv`, `base64`, `logger`, `bigdecimal` from stdlib | Added each as explicit `Gemfile` deps |
| 3 | `wdm` gem fails to build on Ruby 3.4 | Removed from `Gemfile` (Windows-only dev-watch optimisation) |
| 4 | Template heading for Publications was commented out | Broke it out into its own `<section id="publications">` |
| 5 | Template shipped a hidden ClustrMaps block with the upstream author's map id | Deleted in PR #4 |
| 6 | `site.url` with trailing slash → `feed.xml` double-slash | Removed the trailing slash in PR #5 |
| 7 | `jekyll-feed` does not markdownify the subtitle | Split `site.description` (plain) from `site.bio` (markdown) |

## Follow-ups

### Input-dependent (waiting on real values)

- **Google Analytics.** Add a GA4 tracking id to `_config.yml` under
  `google_analytics`. The `<script>` block is already wired and
  conditionally rendered.
- **Arxiv ids for publications.** *iNatSounds MAE* and *Audio
  Geolocation* both list `2505.18726` in `_data/publications.yml`,
  copied faithfully from the pre-migration page. At least one is
  wrong; fix when the correct ids are known.
- **Colour / font tuning.** Current palette: `Google Sans` for
  headings, `Noto Sans` for body, `#3273dc` (Bulma blue) for links,
  `firebrick` for accents. Change in `_sass/_base.scss` and
  `css/index.scss` whenever a new direction is chosen.

### Low-effort polish (can land any time)

- Decide on the footer "Last update: …" line — intentionally skipped
  from the legacy page so it doesn't bit-rot. Add back if desired.
- Hover-to-play mp4 effect for publications that ship an
  `image_mouseover` video (template supports it; no such video is
  currently in `_data/publications.yml`).
- Consider introducing a Projects section if there is anything beyond
  publications worth surfacing.
- Promote the `data-highlight=""` attribute on publications without
  `highlight: true` — either remove it or leave; only matters for the
  filter-buttons UX we dropped in PR #3.

### Infrastructure / dependency hygiene

- **Bump Jekyll to `~> 4.3` (or later).** Would let us drop the
  Ruby 3.4 stdlib shims (`csv`, `base64`, `logger`, `bigdecimal`) —
  Jekyll 4.3+ declares them as deps itself.
- **Deploy via GitHub Actions** instead of the default GitHub Pages
  builder, so `Gemfile.lock` fully controls the production Ruby /
  gem environment rather than Pages' pinned set.
- Keep an eye on the `ffi` and `sass-embedded` native-extension
  gems — any future Ruby major bump is likely to need another
  round of `bundle install` troubleshooting on Windows.

### Not worth chasing (documented for posterity)

- The `bulma.min.css` / `fontawesome.all.min.js` duplication between
  CDN-style and vendored paths. Acceptable for a personal site.
- The legacy page's "(刘武傲)" appeared as `<font size=4>`;
  reproduced via a `.name-chinese` span. Pixel-exact match not
  pursued.
