# Migration Plan: Static HTML → Jekyll Template

## Status

| PR | Title | Status |
|----|-------|--------|
| #1 | Infrastructure — docs + CI | ✅ Merged (`d0574ae`) |
| #2 | Import Jekyll template skeleton | ✅ Merged (`49657a3`) |
| #3 | Migrate content into `_data/` | ✅ Merged (`8df7dfe`) |
| #4 | Switch over — new template goes live | ✅ Merged (`36776b3`) |
| #5 | Personalization and cleanup | ✅ Merged (`0343cfc`, partial — input-dependent items deferred) |

> Retrospective and follow-ups: see [`MIGRATION_SUMMARY.md`](./MIGRATION_SUMMARY.md).

## Goal

Migrate the personal academic website at <https://wuao652.github.io/> from a
plain static HTML site to a Jekyll-based site using the template from
<https://github.com/ngoductuanlhp/ngoductuanlhp.github.io>.

The pre-migration state is preserved on the
[`legacy-backup`](https://github.com/Wuao652/Wuao652.github.io/tree/legacy-backup)
branch.

## PR Plan

### PR #1 — Infrastructure (docs + CI)

- **Goal:** Set up documentation and CI. No visible site changes.
- **Files touched:** `docs/MIGRATION_PLAN.md`, `.github/workflows/jekyll-build.yml`, `README.md`
- **Verify before merging:**
  - CI workflow triggers on PR and passes (the build step is skipped gracefully since no `Gemfile` exists yet).
  - No changes to `index.html`, `stylesheet.css`, `images/`, or `data/`.
- **Rollback:** Revert the merge commit. No impact on the live site.

### PR #2 — Import Jekyll template skeleton

- **Goal:** Add the Jekyll build system and template structure. The template uses the minima theme and has no custom `_layouts/` or `_includes/` — rendering logic lives directly in `index.html` and `cv.html`.
- **Files touched:** `Gemfile`, `Gemfile.lock`, `_config.yml`, `_sass/`, `css/`, `js/`, `assets/`, `.gitignore`
- **Approach for preserving the old site:** Keep the existing `index.html` and `stylesheet.css` as-is. The template's `index.html` is added as `index-new.html` (a non-routed staging copy). The old site continues to render on GitHub Pages. The switchover in PR #4 replaces the old `index.html` with the template version.
- **Verify before merging:**
  - `bundle exec jekyll build` succeeds in CI.
  - The live site at `main` still shows the old `index.html` — no visual change.
- **Rollback:** Revert the merge commit. The site falls back to the old static HTML (GitHub Pages serves `index.html` without Jekyll if no `_config.yml` exists).

### PR #3 — Migrate content into `_data/`

- **Goal:** Convert existing bio, publications, and project content into structured YAML/JSON files under `_data/` so the template can render them.
- **Files touched:** `_data/*.yml` (`publications.yml`, `authors.yml`, `experience.yml`, `news.yml`, `education.yml`, `services.yml`), `index-new.html`, `cv.html`
- **Verify before merging:**
  - `bundle exec jekyll build` succeeds.
  - Running `bundle exec jekyll serve` locally shows the template rendering the migrated content correctly.
- **Rollback:** Revert the merge commit. Template still builds but shows placeholder or empty content.

### PR #4 — Switch over

- **Goal:** The new template becomes the live site. Remove or archive old `index.html` and `stylesheet.css`.
- **Files touched:** `index.html` (replaced by template version), `stylesheet.css` (removed), `_config.yml` (update excludes)
- **Verify before merging:**
  - The local preview matches the expected design with all content.
  - All images load correctly from `images/`.
  - No broken links.
- **Rollback:** Revert the merge commit. Old `index.html` and `stylesheet.css` are restored, site reverts to the pre-migration look.

### PR #5 — Personalization and cleanup

- **Goal:** Customize colors, fonts, SEO metadata, analytics, and remove any leftover template placeholder content.
- **Files touched:** `_sass/` partials, `_config.yml`, `_data/` tweaks, `index.html`
- **Verify before merging:**
  - Visual review of the final site.
  - Lighthouse or similar audit for SEO and performance.
  - No template placeholder text remaining.
- **Rollback:** Revert the merge commit. Site reverts to default template styling from PR #4.
