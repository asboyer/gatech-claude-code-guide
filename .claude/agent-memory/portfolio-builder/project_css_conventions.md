---
name: CSS tokens and conventions
description: CSS custom property names, spacing scale, breakpoints, and class naming patterns used in css/style.css
type: project
---

## Color tokens (Gruvbox light/dark via prefers-color-scheme)
- `--bg`, `--bg-soft`, `--bg-alt` — background hierarchy
- `--fg`, `--fg-muted`, `--gray` — foreground hierarchy
- `--accent` (= `--aqua`) — primary accent; used for links, focus rings, active nav
- `--red`, `--green`, `--yellow`, `--blue`, `--purple`, `--aqua`, `--orange` — named palette colors

## Typography
- `--font-mono` — system monospace stack; used for everything (no web fonts)
- `--line: 1.6` — line-height

## Spacing scale
- `--space-xs: 0.25rem`, `--space-sm: 0.5rem`, `--space-md: 1rem`, `--space-lg: 2rem`, `--space-xl: 4rem`

## Layout tokens
- `--measure: 70ch` — max content width
- `--border: 1px solid var(--bg-alt)` — standard divider
- `--radius: 0` — no border-radius; sharp corners throughout

## Breakpoints (mobile-first)
- `≥640px` — tablet; h1/h2 font sizes increase; main padding switches to `--space-xl`
- `≥960px` — desktop; `.container` padding increases to `--space-lg`
- `≥1024px` — used in `.projects-grid` for full 3-column layout

## Class naming patterns
- Shared components: `.site-nav`, `.site-nav__brand`, `.site-nav__links`, `.site-footer`, `.skip-link`, `.container`
- Resume page: `.resume-header`, `.resume-section`, `.resume-entry`, `.resume-entry__header`, `.resume-entry__org`, `.resume-entry__date`, `.resume-entry__title`, `.resume-entry__bullets`, `.resume-skills`, `.resume-skills__group`
- Projects page: `.projects-section`, `.projects-grid`, `.projects-card`, `.projects-card__header`, `.projects-card__title`, `.projects-card__date`, `.projects-card__desc`, `.projects-card__tags`, `.projects-card__tag`, `.projects-card__links`, `.projects-card__link`
- CSS sections: 1=Reset, 2=Tokens, 3=Base, 4=Layout, 5=Shared components, 6=Home (Phase 1), 7=Resume (Phase 2), 8=Projects (Phase 3), 9=Responsive

## Focus style (must match across all interactive elements)
`outline: 2px solid var(--accent); outline-offset: 2px;`

**Why:** Set in Phase 0 as the global `:focus-visible` standard. All new interactive elements must use this exact rule.
**How to apply:** Every new `<a>`, `<button>`, or focusable element added to the site must include this focus rule in its CSS block.
