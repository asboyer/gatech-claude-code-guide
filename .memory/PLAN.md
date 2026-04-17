# Project Plan — Personal Website

## Overview
A three-page personal website (home, resume, projects) built with pure HTML and CSS — no JavaScript, no frameworks, no build tools — hosted on GitHub Pages.

## Phases

### Phase 0 — Scaffolding & Base Files
**Goal:** Set up the repo structure, shared CSS, and the HTML boilerplate that all pages will use.
**Delivers:**
- Folder structure (`/css`, `/assets` or similar — you decide, log it in DECISIONS.md)
- `style.css` with CSS reset/normalize, CSS custom properties for colors/fonts/spacing, responsive base styles, and shared component styles (nav, footer)
- Boilerplate HTML that each page will copy: doctype, meta tags, nav, main, footer
- A working `index.html` that renders the boilerplate with placeholder content

**Done when:** You can open `index.html` in a browser and see styled nav, a placeholder main area, and a footer. The page is responsive.

### Phase 1 — Home Page
**Goal:** Build out `index.html` into a real landing page.
**Depends on:** Phase 0
**Delivers:**
- Bio/intro section
- Navigation to Resume and Projects pages
- Social or contact links
- Any hero or visual treatment
- Mockup Contact Me form that is not functional but has fields to be filled out and a button to submit the form

**Done when:** `index.html` is complete, responsive, and links point to `resume.html` and `projects.html` (even if those don't exist yet).

### Phase 2 — Resume Page
**Goal:** Build `resume.html`.
**Depends on:** Phase 0 (independent of Phase 1 and Phase 3)
**Delivers:**
- Sections for experience, education, skills, and anything else relevant
- Clean layout that prints well (add a `@media print` block in CSS if needed)
- Semantic, accessible markup

**Done when:** `resume.html` renders a full resume, is responsive, and produces a clean printout.

### Phase 3 — Projects Page
**Goal:** Build `projects.html`.
**Depends on:** Phase 0 (independent of Phase 1 and Phase 2)
**Delivers:**
- A card or section layout for individual projects
- Each project shows: title, description, tech stack, and link(s)
- Responsive grid or flex layout for the cards

**Done when:** `projects.html` shows at least 2-3 placeholder projects, is responsive, and styled consistently with the rest of the site.

## Phase Dependency Map

Phase 0 is the foundation. Phases 1, 2, and 3 can be done in any order after Phase 0 is complete. They are independent of each other.

```
Phase 0 ──┬── Phase 1 (Home)
          ├── Phase 2 (Resume)
          └── Phase 3 (Projects)
```

## Working Agreements
- One phase per session. Don't bleed into the next phase.
- Log every structural or design decision in `.memory/DECISIONS.md`.
- Update `.memory/STATUS.md` at the end of each session with the current phase and what changed.
- Update `.memory/LAST_SESSION.md` at session close with: what was done, what's next, and any open questions.
