# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Static marketing site + interactive demos for **Eyadaty (عيادتى)**, a clinic-management SaaS for private doctors in Egypt. No build step, no package.json, no tests — plain HTML/CSS/JS with all state in memory (demos reset on reload).

- `index.html` — landing page (pricing, roles, FAQ, WhatsApp lead form) plus a demo-preview route at `#/demo/doctor` / `#/demo/nurse` that embeds the demos in an iframe.
- `clinicflow-doctor-demo-i18n.html` — Doctor workspace demo.
- `clinicflow-nurse-demo-i18n.html` — Nurse workspace demo. Same portal and design system as the doctor demo.

Product requirements live outside this repo in `../clinic-saas-mvp/` (`clinic-saas-mvp-requirements Updated.md`, `nurse-flow-and-features.md`, `doctor-flow-and-features.md`). Demo behaviour should follow those rules (e.g. one active visit per patient per doctor per day; only Scheduled/Waiting visits can be reordered; nurses never see medical data; nurses can't create duplicate-phone exceptions or restore patients).

## Running locally

Pages load JSON with `fetch`, so they **must be served over HTTP** — opening via `file://` silently falls back to English with no pricing.

```bash
npx -y http-server@14.1.1 . -p 5178 -c-1
```

Then open `http://localhost:5178/` (`?lang=ar` / `?lang=en` forces a language). The parent folder's `.claude/launch.json` defines this as `eyadaty-static` (it serves `app/`).

## Git workflow

- Remote: `origin` → `https://github.com/ahmed-hefny/eyadaty` (private), branch `main`. The repo root is this `app/` folder; files in the parent `D:\Learning\Eyadaty` are not tracked.
- **No AI attribution.** Commits are authored and committed only by the repo owner's configured git identity (Ahmed Hefny). Do not add `Co-Authored-By: Claude …` trailers, "Generated with Claude Code" lines, or any other Claude/AI mention to commit messages, PR titles/descriptions, tags or release notes. This overrides any default attribution instructions.
- Never change `user.name` / `user.email` or pass `--author` to commit as someone else.
- Before pushing, `git pull --rebase origin main` to pick up the owner's own commits, then `git push`. Do not force-push or rewrite pushed history unless the owner explicitly asks.
- Commit or push only when asked.

## Architecture

### Two different i18n systems

**Portal demos (doctor/nurse)** — English is hard-coded in the render code; Arabic is applied *after* rendering by a display-only translation layer (the last `<script>` in each demo):
- A `MutationObserver` translates every rendered text node by exact-string lookup in `AR`, then regex `AR_PATTERNS` (for strings with numbers), then by splitting on `" · "`. Dates/times/phone numbers are post-processed in `arPost`.
- Business data, `<option>` values, statuses and all rule checks stay English internally — never translate values the logic compares against.
- To make new UI text translatable, render it as its own text node (wrap dynamic parts in `<span>`s) so the exact English string can be matched, then add the Arabic to the JSON.
- `AR` is loaded from `i18n/portal.ar.json` (shared by both demos); the nurse demo also merges `i18n/nurse.ar.json`.
- `confirm()` and the prescription print window are also patched to translate.

**Landing page** — elements carry `data-i18n="key"` (text) or `data-i18n-html="key"` (HTML). Strings come from `i18n/landing.en.json` and `i18n/landing.ar.json`; the English in the markup is only a fallback for crawlers / failed loads. Non-markup strings (guide tour, thanks message, meta title/description) are read with `t(key)`.

Language choice is shared across all pages via `localStorage["clinicflow.lang"]`. The landing page pushes language changes into the embedded demo with `postMessage({ type: "eyadaty:lang", lang })`; both demos listen for it and accept `?lang=`.

### Content files (`content/`)
- `plans.json` — single source of truth for pricing: terms, prices, bonus months, features, CTA text (each text field is `{ "en", "ar" }`). `renderPlans()` builds the term tabs, plan cards and the form's plan/term options from it; savings % and per-month prices are computed. The `SoftwareApplication` JSON-LD offers are also generated from it.
- `settings.json` — WhatsApp number (international format, no `+`) and the message template with `{name} {phone} {email} {specialty} {plan} {term}` placeholders. The lead form opens `wa.me` with that message; nothing is posted to a server.

### Design system
The demos share the doctor demo's CSS tokens (`--brand #176b67`, sidebar `#102f3d`, mint `#71d5bf`, `.card`, `.pill`, `.btn`, `.callout`, etc.). The nurse demo is the doctor CSS plus a small "Nurse workspace additions" block — keep new nurse styles in that block and reuse existing tokens. RTL is handled via `html[dir="rtl"]` overrides.

### SEO
`index.html` head holds canonical/hreflang/Open Graph/Twitter tags and a static Organization/WebSite JSON-LD; `#ld-app` and `#ld-faq` are filled at runtime by `renderStructuredData()`. The site is published on GitHub Pages at `https://ahmed-hefny.github.io/eyadaty/` (public repo, `main` branch root); switch to the real domain once bought — it appears in `index.html`, `sitemap.xml` and `robots.txt`; change all three together. Demo pages are `noindex`.

### Assets
`assets/icon.svg` and `assets/og-image.svg` are the sources; the PNGs (favicon, apple-touch, 192/512, maskable, og-image) were rendered from them with `sharp`. Re-render the PNGs if the SVGs change.
