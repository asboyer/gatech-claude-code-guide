## Part 1 — Set Up the Working Memory

Create the following directory structure at the project root:

```
.memory/
  sessions/
  PLAN.md          ← you'll write this (see Part 3)
  STATUS.md
  DECISIONS.md
  LAST_SESSION.md
.claude/
  tasks/
```

Then populate the tracking files:

**`.memory/STATUS.md`**
```
# Project Status

Stack: HTML / CSS (no JS, no frameworks)
Hosting: GitHub Pages
Build command: none (static files)
Test command: none
Lint command: none
Git: yes
Initialized: <today's date>
Current phase: 0 — Scaffolding
```

**`.memory/DECISIONS.md`**
```
# Decisions

<!-- Append new decisions at the bottom. One entry per decision. Format: -->
<!-- ## D-001: <title> -->
<!-- **Context:** why this came up -->
<!-- **Decision:** what we chose -->
<!-- **Reasoning:** why -->
```

**`.memory/LAST_SESSION.md`**
```
# Last Session

No sessions yet.
Updated each time a session closes with: what was done, what's next, and any blockers.
```

Do NOT create any project files yet. Just the memory structure and these three files.

---

## Part 2 — What We're Building

We are building a **personal website** with the following constraints:

- **Pure HTML and CSS only.** No JavaScript. No frameworks. No build tools.
- **Hosted on GitHub Pages** — everything lives in the repo root or a `docs/` folder.
- **Three pages:**
  1. **Home** (`index.html`) — a landing page with a short bio, links to the other pages, and any social/contact links.
  2. **Resume** (`resume.html`) — a web version of a resume/CV. Should be clean, readable, and printable via the browser's print function.
  3. **Projects** (`projects.html`) — a showcase of projects. Each project gets a card or section with a title, description, tech used, and a link (GitHub, live demo, etc.).
- **Shared layout** — all pages share a common header/nav and footer. Use a consistent `style.css`.
- **Responsive** — should look good on desktop and mobile. Use media queries, not a CSS framework.
- **Accessible** — semantic HTML (`<nav>`, `<main>`, `<section>`, `<footer>`), proper heading hierarchy, alt text on images, sufficient color contrast.

---

## Part 3 — Write the Plan

Now write **`.memory/PLAN.md`** — this is our north-star document. Structure it exactly like this:

```markdown
# Project Plan — Personal Website

## Overview
<1-2 sentences restating what we're building and the core constraints>

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

