# Decisions

<!-- Append new decisions at the bottom. One entry per decision. Format: -->
<!-- ## D-001: <title> -->
<!-- **Context:** why this came up -->
<!-- **Decision:** what we chose -->
<!-- **Reasoning:** why -->

## D-001: Folder structure
**Session:** 2026-04-15-0157-scaffolding.md
**Context:** Phase 0 needed a layout that supports a single shared stylesheet, page-scoped configs, and an asset bucket.
**Decision:** `css/`, `assets/`, and `config/` directories at the project root. Single `css/style.css`. Three HTML files at the root (`index.html`, `resume.html`, `projects.html`).
**Reasoning:** Flat root keeps GitHub Pages defaults working with zero config. `config/` separates content from markup so the future resume-loader phase has a clean target. `assets/` tracked via `.gitkeep` so it exists even while empty.

## D-002: Typography — system monospace, no web fonts
**Session:** 2026-04-15-0157-scaffolding.md
**Context:** Design direction is "developer aesthetic, monospace." Web fonts (Google Fonts, etc.) would add a network request and are unnecessary given the look we want.
**Decision:** Use a system monospace stack: `ui-monospace, "SF Mono", Menlo, Monaco, "Cascadia Mono", Consolas, "Liberation Mono", monospace`. No `<link>` to any external font CDN.
**Reasoning:** Zero network cost, no FOUT, looks native on every OS, matches the developer-tool feel. Aligns with the "no build tools, no frameworks" project rules.

## D-003: Theme — Gruvbox light + dark via prefers-color-scheme
**Session:** 2026-04-15-0157-scaffolding.md
**Context:** User wanted both light and dark mode support. Project rule: no JavaScript.
**Decision:** Define Gruvbox light tokens in `:root`, override them inside `@media (prefers-color-scheme: dark)` with the Gruvbox dark palette. Add `<meta name="color-scheme" content="light dark">` to every page so form controls and scrollbars also follow the OS theme. No theme toggle button.
**Reasoning:** A toggle would require JS. `prefers-color-scheme` is the only no-JS path to dual-theme support, and it follows OS settings — which is what most users actually want anyway. Gruvbox has well-defined paired light/dark palettes that map 1:1.

## D-004: Scaffold all three HTML files in Phase 0 for parallel work
**Session:** 2026-04-15-0157-scaffolding.md
**Context:** Phases 1 (home), 2 (resume), and 3 (projects) are independent per `PLAN.md` and could be worked in parallel — but only if the boilerplate already exists for all three pages.
**Decision:** Create `index.html`, `resume.html`, and `projects.html` in Phase 0, each with the full shared boilerplate (nav, footer, skip link, `aria-current` set correctly). `style.css` ships with labeled empty page-section blocks (`6. PAGE SECTIONS — HOME`, `7. RESUME`, `8. PROJECTS`) so each phase owns exactly one CSS block and one HTML file.
**Reasoning:** Without this, two parallel sessions on Phases 2 and 3 would both edit `style.css` in undefined regions and merge-conflict. The labeled blocks make the ownership contract explicit. Cost is minimal — empty `<main>` with a TODO comment.

## D-006: Resume page includes name + contact header
**Session:** 2026-04-16-0000-resume-page.md
**Context:** The resume PDF had a name/contact block at the top. Question was whether to include it on the web page since the nav already shows the name.
**Decision:** Include the header — name, phone, email, and LinkedIn — centered above the sections.
**Reasoning:** The page may be printed or screenshotted directly (bypassing the nav), so the header makes the resume self-contained. Mirrors standard resume convention.

## D-007: Projects section excluded from resume page
**Session:** 2026-04-16-0000-resume-page.md
**Context:** The resume PDF includes a "Projects & Research" section (RoboJackets, GT IVA Lab).
**Decision:** Exclude projects from `resume.html`. They will be covered in Phase 3 on `projects.html`.
**Reasoning:** Avoids duplicating content across two pages. The projects page is the canonical home for project detail; the resume page focuses on education, experience, and skills.

## D-008: `<dl>` used for Technical Skills
**Session:** 2026-04-16-0000-resume-page.md
**Context:** Skills are grouped by category (Programming Languages, Frameworks, AI/ML) with a list of items per category.
**Decision:** Use `<dl>` with `<div class="resume-skills__group">` wrappers, `<dt>` for category name, `<dd>` for the items string.
**Reasoning:** A description list is the correct semantic element for term-definition pairs. More meaningful to screen readers than a nested `<ul>`, and renders inline naturally with flex layout.

## D-005: Per-page configs as the single source of truth for content
**Session:** 2026-04-15-0157-scaffolding.md
**Context:** User asked for user-identifying info to live in a config file so a future resume-loader phase can swap real data in without rewriting HTML. Then refined: each page should have its own config.
**Decision:** Three config files: `config/home.json`, `config/resume.json`, `config/projects.json`. Each is the source of truth for its page. `home.json` additionally owns the shared identity (`identity.name`, `identity.name_display`, `identity.year`) used in nav, footer, and `<title>` on every page. Because there is no JavaScript and no build step, configs cannot be loaded at runtime — they act as a documented contract that humans (or a future loader) keep in sync with the HTML. Placeholder identity for Phase 0: George P. Burdell.
**Reasoning:** Externalizes content from markup with zero runtime cost. When the resume-loader phase lands, these files become its inputs — the swap is mechanical instead of a search-and-replace through HTML. Per-page (not one mega-config) keeps Phase 1/2/3 owners from stepping on each other.
