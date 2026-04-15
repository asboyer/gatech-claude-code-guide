---
name: smoke-test
description: Run a smoke test on the personal website project after a phase wraps or any significant change. Verifies file structure, HTML/CSS rules, config sync, accessibility basics, and cross-page consistency, then produces an actionable report grouped by FAIL / WARN / PASS. Use this skill whenever the user says "smoke test", "/smoke-test", "verify the site", "check things still work", "end of phase check", "did I break anything", or finishes a phase, makes structural edits to HTML/CSS/config, or otherwise needs confidence the site still functions before committing or moving on.
---

# Skill: smoke-test

## Purpose
A static HTML/CSS site has no compiler and no test runner. The only thing standing between a clean phase and a broken one is a checklist someone actually runs. This skill is that checklist — fast, scoped to this project's rules (see `CLAUDE.md`), and ends in a report with concrete fixes the user can act on.

## When to run
- End of any phase (Phase 0, 1, 2, 3 — and any future ones).
- After significant edits to `css/style.css`, any `*.html`, or any `config/*.json`.
- Before committing or merging work.
- Anytime the user wants confidence things still render and follow the project rules.

## What this skill does NOT do
- It does not start a dev server or click around in a browser. Those are *manual* steps it tells the user to do at the end.
- It does not auto-fix anything. It reports — the user (or you in a follow-up turn) decides what to change.
- It does not validate against the W3C — it validates against *this project's* rules in `CLAUDE.md`.

## Inputs
- Optional: a phase number or scope hint (e.g. `phase 0`, `home page`, `css only`). If given, weight the report toward those areas but still run the full check — phases are cheap to recheck and cross-cutting bugs are easy to miss.

## Steps

Run the checks below in order. Use `Read`, `Glob`, and `Grep` — never modify files. Capture findings as you go, then produce the report at the end.

### 1. Project rules refresher
Read `CLAUDE.md` once at the start. The rules there are the source of truth — if a rule was added since this skill was written, the skill should still respect it. If a check below contradicts `CLAUDE.md`, trust `CLAUDE.md` and note the drift in the report.

### 2. File structure
Confirm the expected layout exists. Missing files for phases that haven't started yet are *expected*, not failures — use the phase context to decide.

Required at all times after Phase 0:
- `index.html`, `resume.html`, `projects.html`
- `css/style.css`
- `config/home.json`, `config/resume.json`, `config/projects.json`
- `assets/` (directory, may be empty)
- `.memory/` with `PLAN.md`, `STATUS.md`, `DECISIONS.md`, `LAST_SESSION.md`

### 3. HTML rules (run on every `*.html` at the project root)
For each HTML file, verify:

- **No JavaScript.** Grep for `<script` (case-insensitive). Any hit → FAIL.
- **No inline styles.** Grep for `style=` and any inline `<style>` blocks. Any hit → FAIL.
- **One stylesheet only.** Exactly one `<link rel="stylesheet">` and it points to `css/style.css`. More than one, or a different path → FAIL.
- **Doctype + lang.** First non-blank line is `<!DOCTYPE html>`. The `<html>` tag has `lang="en"` (or another explicit lang). Missing → FAIL.
- **Viewport meta.** `<meta name="viewport" content="width=device-width, initial-scale=1">` present → required for responsive. Missing → FAIL.
- **color-scheme meta.** `<meta name="color-scheme" content="light dark">` present → required for the OS theme switch to work cleanly. Missing → WARN.
- **Title set.** `<title>` exists and is non-empty. Missing/empty → FAIL.
- **Semantic landmarks.** Has `<header>`, `<nav>`, `<main>`, `<footer>`. Missing any → WARN (FAIL if `<main>` is missing — it's the skip-link target).
- **Single h1.** Exactly one `<h1>` per page. Zero or more than one → WARN.
- **Image alts.** Every `<img>` has a non-empty `alt=` attribute (decorative images may use `alt=""` — that's still present, just empty, and is OK). Missing attribute entirely → FAIL.
- **Skip link.** `<a class="skip-link" href="#main">` exists and points to an element with `id="main"`. Missing or broken target → WARN.
- **Nav active state.** Exactly one nav link has `aria-current="page"`, and it's the link that points to the current page. Wrong link or none → WARN.
- **Internal links resolve.** Every `href` that isn't external (no `http`, no `mailto:`, no `#`) points to a file that exists at that path. Broken link → FAIL.

### 4. CSS rules (`css/style.css`)
- File exists and is non-empty. Missing → FAIL.
- Has the labeled section banners for sections 1–9 from the Phase 0 plan (RESET, TOKENS, BASE, LAYOUT, COMPONENTS — SHARED, PAGE SECTIONS — HOME / RESUME / PROJECTS, RESPONSIVE). Missing the per-page banners is what makes parallel phase work safe — without them, Phases 1/2/3 will collide. Missing any → WARN.
- `:root` defines the Gruvbox light tokens. `@media (prefers-color-scheme: dark)` exists and overrides them. Missing the dark block → FAIL (dark mode is a stated design goal).
- No raw hex colors outside `:root` and the dark `@media` block. Grep for `#[0-9a-fA-F]{3,6}` outside those two blocks → WARN with the line numbers (the rule is "all colors via custom properties").
- At least one mobile-first media query (`@media (min-width:`) exists in section 9. Missing → WARN.
- Brace balance: count `{` vs `}` — they must match. Mismatch → FAIL with the likely region.

### 5. Config files (`config/*.json`)
- Each file parses as valid JSON. Use `python3 -c 'import json,sys; json.load(open(sys.argv[1]))' <path>` (or read + mentally parse — but the python check is faster and definitive). Parse error → FAIL with the line/column from the parser.
- `config/home.json` has `identity.name`, `identity.name_display`, `identity.year`, `hero.headline`, `hero.tagline`. Missing key → FAIL.
- `config/resume.json` has at least `summary`, `experience`, `education`, `skills` (may be empty). Missing key → WARN (Phase 2 may not have started).
- `config/projects.json` has at least `projects`. Missing → WARN.

### 6. Config-to-HTML sync
This is the thing most likely to silently rot, because the site has no runtime template engine.

For each value below, read it from the config and grep the HTML to confirm a match. Mismatch → FAIL with both values shown side by side.

- `home.identity.name` appears in every page's `<title>` and footer copyright.
- `home.identity.name_display` appears as the `.site-nav__brand` text on every page.
- `home.identity.year` appears in every page's footer copyright.
- `home.hero.headline` appears as the `<h1>` in `index.html`.
- `home.hero.tagline` appears in the `index.html` placeholder paragraph (only if Phase 1 hasn't replaced it — if `index.html` has clearly moved past placeholder content, downgrade to WARN).

### 7. Cross-page consistency
- All HTML files have the same set of nav links in the same order.
- All HTML files have the same footer block.
- All HTML files reference `css/style.css` at the same relative path.
Differences → WARN with a diff-style summary.

### 8. Memory hygiene (light touch — don't fail the build on this)
- `.memory/STATUS.md` `Current phase` line matches the phase the user said they just finished, or is one ahead. Mismatch → WARN ("STATUS.md says phase X, you said you finished Y — was the session closed?").
- `.memory/sessions/` has no open sessions older than today. Old open sessions → WARN.

### 9. Manual checks the skill cannot do
List these in the report under a **"Run these yourself"** heading. Don't try to do them automatically — they need a real browser and a human eyeball.

1. Open `index.html`, `resume.html`, `projects.html` directly in a browser (`open index.html` on macOS) and confirm each renders nav + main + footer with no console errors.
2. Resize the window narrow (≤375px) and confirm no horizontal scroll, nav still works, text still readable.
3. Toggle the OS appearance between Light and Dark — the page should swap palettes immediately, no flicker, both modes legible.
4. Tab through the page from the address bar — the skip link should appear on first Tab, every nav link should show a visible focus outline.
5. (Phase 2+) Print preview `resume.html` — should look clean, no nav/footer bleeding into the print layout if a `@media print` block exists yet.

## Report format

Always print the report with this exact structure so it's scannable. Group by severity, lead with the worst.

```
# Smoke Test — <date> — <phase context if given>

## ❌ FAIL  (must fix before moving on)
- <one line summary> · <file:line if known>
  Fix: <concrete action — what to change, where>

## ⚠️  WARN  (should fix soon, won't block)
- <one line summary> · <file:line if known>
  Fix: <concrete action>

## ✅ PASS  (checks that ran clean)
- <category>: <count> checks passed
- ...

## 🖐  Run these yourself
1. <manual step>
2. <manual step>
...

## Summary
<N> fail, <N> warn, <N> pass categories. <One sentence verdict: ready to commit / fix the FAILs first / etc.>
```

### Why this format
- **Severity-grouped, not file-grouped:** the user wants to know "what do I have to fix" before "what file is it in". File paths still appear, just inside each finding.
- **Every finding has a Fix line:** the user asked for *actionable* steps. A finding without a fix is just complaining.
- **Manual steps are separated:** so the user knows what's left for them after this skill runs and doesn't think they're done when they're not.
- **One-sentence verdict at the end:** lets the user act in 2 seconds without re-reading the whole report.

## Rules for running this skill
- Read-only. Never edit a file from this skill — even an obvious one-line fix. The user runs the skill specifically to *see* the state, not to have it changed silently.
- Don't run the dev server, don't open a browser, don't modify git state.
- If a check is ambiguous (e.g. "is this still placeholder content?"), prefer WARN over FAIL — false positives are worse than false negatives in a smoke test, because they train the user to ignore the report.
- If a whole category can't be checked (e.g. `resume.html` doesn't exist yet because Phase 2 hasn't started), say so explicitly under a **"Skipped"** subsection rather than silently passing.
- Keep the report tight. If 40 things passed, it's "HTML rules: 12/12 passed", not 12 bullet points.
