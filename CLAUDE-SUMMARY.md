# Site refresh — summary

A refresh of engineering-manager.com: modernized the build, cleaned up dead
integrations, and re-skinned the site in a refined editorial/typographic style.
All blog post content, URLs, and cross-links are preserved unchanged.

Done on branch `refresh-site-2026`.

## Build modernized (Jekyll 3.9 → 4.4.1)

- `Gemfile` now pins Jekyll 4 + plugins directly (dropped the `github-pages` gem
  lock and the old `faraday`/`base64`/`bigdecimal`/`csv` pins). `Gemfile.lock`
  regenerated.
- Local toolchain: installed **rbenv + Ruby 3.3.0** (matches `.ruby-version`),
  since only system Ruby 2.6 was present.
- `.github/workflows/pages.yml` builds with Jekyll 4 + Dart Sass and deploys to
  GitHub Pages via Actions. Hosting/DNS unchanged.

## Integrations cleaned up

- `_includes/analytics.html`: replaced dead Universal Analytics with **GA4**,
  gated to a configured Measurement ID *and* `production` only.
- Removed Mailchimp newsletter, Disqus comments, HubSpot tracking, and the unused
  `intercom.html` include (plus the Mailchimp CSS block in `head.html`).

## Editorial / typographic re-skin

- Design tokens as CSS custom properties (warm paper, near-black ink, terracotta
  accent), with an automatic dark mode via `prefers-color-scheme`.
- Fonts: Fraunces (display) + Newsreader (reading).
- New home page (`index.md` + `_layouts/home.html`): eyebrow, large serif name,
  lede, bio with the Intercom · HubSpot · Fin career arc, social pills
  (LinkedIn / GitHub / HuggingFace as inline SVGs), and a "Selected writing" list
  driven by a `featured:` list in `_config.yml`.
- Restyled nav, footer, post layout (SVG share row, "Keep reading" related cards),
  and blog catalogue + pagination.

## Verified

- Clean production build; all 30 posts generate at their original permalinks.
- `/blog/`, `/blog/2`, `/blog/3` pagination, `feed.xml`, `sitemap.xml` intact.
- GA4 absent unless an ID is set in production.
- No leftover Disqus / Mailchimp / HubSpot / UA references.

## Follow-ups for the owner

1. Repo Settings → Pages → Source → **GitHub Actions** (required for the workflow).
2. Set `ga4_measurement_id` in `_config.yml` (empty = analytics off).
3. Confirm / swap the `featured:` posts on the home page.
4. Confirm the HuggingFace URL (currently guessed as `huggingface.co/jniechcial`).
5. Bio copy is a draft anchor — refine, especially the HubSpot/Fin lines.
6. Pre-existing bug: two posts' `related` links point at `2018-12-3/...` but the
   page is at `2018-12-03/...` (zero-padded) — 404 in the old site too.

## Local development

```bash
bundle install
bundle exec jekyll serve   # http://localhost:4000
```
