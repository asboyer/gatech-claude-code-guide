## Project
Static personal website — pure HTML and CSS, hosted on GitHub Pages. No JavaScript. No frameworks. No build tools.

## Structure
```
index.html          → Home page
resume.html         → Resume
projects.html       → Project showcase
css/style.css       → All styles, shared across pages
assets/             → Images and static files
.memory/            → Project memory (status, decisions, sessions)
.claude/tasks/      → Skills (start-session, close-session)
```

## Rules
- **No JS.** If you think you need it, you don't. Solve it with CSS.
- **One shared stylesheet.** No inline styles, no per-page CSS files.
- **Semantic HTML.** Use `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`. No `<div>` soup.
- **Responsive.** Mobile-first media queries. No CSS frameworks.
- **Accessible.** Proper heading hierarchy, alt text on all images, sufficient color contrast.

## Session Workflow
1. Start every session with: `/start-session <goal>`
2. Do the work.
3. End every session with: `/close-session <summary>`

Never skip these. The `.memory/` directory is how we maintain continuity between sessions.

## Memory Files
- `.memory/PLAN.md` — the phases and "done when" criteria. Read before working.
- `.memory/STATUS.md` — current phase and last update. Don't edit manually.
- `.memory/DECISIONS.md` — architectural/design choices. Append-only.
- `.memory/LAST_SESSION.md` — what happened last time. Overwritten each close.

## Conventions
- CSS custom properties for all colors, fonts, and spacing. No magic numbers.
- Class names: kebab-case (`.hero-section`, `.project-card`).
- Commit messages: `phase-<N>: <what changed>` (e.g., `phase-0: add base styles and nav`).
