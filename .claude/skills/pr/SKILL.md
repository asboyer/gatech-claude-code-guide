---
name: pr
description: Commit all uncommitted changes from the current session as conventional commits (one key change per commit), create a new branch, push it, and open a pull request with a detailed description. Takes an optional positional argument — pass "merge", "auto", or "auto merge" to enable GitHub auto-merge with squash strategy and switch back to master after; pass nothing to just open the PR and stay on the branch. Use this skill whenever the user says "/pr", "open a pr", "ship it", "make a pr", "commit and push and pr", "wrap this up", "send it", "pr this", or otherwise indicates they want session work committed and proposed for merge. Use this even if the user only says "pr" — that's the trigger.
---

# Skill: pr

## Purpose
End-of-session shipping ritual: take everything the user just did, slice it into clean conventional commits, put it on a feature branch, push it, and open a PR with a description that actually tells a reviewer (and future-them) what changed and why. Optionally: auto-merge it once checks pass and drop them back on master ready for the next session.

This skill exists because the user shouldn't have to remember the seven-step git/gh dance every time. They say "pr" and the work ships.

## Inputs

The skill takes one optional positional argument:

| Argument value           | Behavior                                                                  |
|--------------------------|---------------------------------------------------------------------------|
| _(none)_                 | Commit, push, open PR. Stop. User stays on the feature branch.            |
| `merge` / `auto` / `auto merge` | Same as above, then enable GitHub auto-merge (squash), wait for it to land or queue, switch back to `master`, and pull. |

Treat the argument case-insensitively. Anything else is unrecognized — ask the user what they meant rather than guessing.

## Preflight

Before doing anything destructive, confirm the environment is sane. Each of these is a hard stop — fix or ask, don't push through.

1. **Inside a git repo.** `git rev-parse --is-inside-work-tree` returns `true`.
2. **`gh` is installed and authed.** `gh auth status` succeeds. If not, tell the user to run `gh auth login` and stop.
3. **There are changes to ship.** `git status --porcelain` is non-empty. If empty, tell the user "nothing to commit — already clean" and stop. Don't open an empty PR.
4. **Currently on `master`** (or whatever the project's default is — `git remote show origin | grep "HEAD branch"` confirms). If the user is already on a feature branch, ask whether to push that one as-is or branch off it. Don't silently rebase.
5. **`master` is up to date with origin.** `git fetch origin master` then check if local `master` is behind. If behind, tell the user and ask whether to pull first — a stale base makes a noisy PR.
6. **No secrets staged.** Quick scan of changed files for filenames like `.env`, `*.key`, `credentials*`, `*.pem`, and grep for obvious patterns (`AKIA`, `sk-`, `ghp_`, `BEGIN PRIVATE KEY`). If anything matches, stop and warn — never include these in a commit without explicit user confirmation.

## Session close (run before any git work)

Why this comes first: if the session has memory edits (`.memory/STATUS.md`, `.memory/DECISIONS.md`, `.memory/LAST_SESSION.md`, the open session file under `.memory/sessions/`) that haven't been written yet, running the PR flow first would commit a half-closed session — and the close itself would then create a *second* batch of uncommitted edits, forcing a follow-up commit/PR. We close first so the close edits are part of the same PR.

Steps:

1. **Detect an open session.** List `.memory/sessions/` and look for any file whose contents include `Status: 🟢 Open` (or any non-`✅ Closed` status). If none, skip the rest of this section and go straight to "Steps".
2. **Run the project's close-session flow.** This project ships a `close-session` skill (also reachable as `c-sesh` / `/c-sesh`). Invoke it with a short summary of what shipped this session — pull the summary from the work log on the open session file plus what the diff in step 1 of "Steps" reveals. The close-session skill is responsible for updating `STATUS.md`, appending to `DECISIONS.md`, overwriting `LAST_SESSION.md`, and marking the session file Closed.
3. **Re-run preflight items 3 (changes exist) and 6 (no secrets)** after close, since close-session just touched the working tree. The new memory edits should now appear in `git status` — that's expected and they ride along in the PR.
4. If close-session fails or asks the user a question, stop and surface it. Don't paper over it to keep the PR moving — a half-written session memory is worse than a delayed PR.

This applies to **both** the default PR flow and the `auto` variant. The reasoning (avoid a second uncommitted batch) is identical either way.

## Steps

### 1. Survey the changes
Run these in parallel:
- `git status --porcelain` — full list of changed/new files
- `git diff` — unstaged tracked changes
- `git diff --staged` — anything already staged
- `git log master..HEAD --oneline` — anything already committed on this branch (if not on master)
- `git log -10 --oneline` — recent history, to match commit message *style* (capitalization, scope conventions)

Read enough of the diffs to actually understand what changed. Don't skim. The commit messages and PR description depend on this — vague messages are the #1 thing the user is paying you to avoid.

### 2. Group changes into logical commits
The rule the user gave: **one key change per commit.** A "key change" is one coherent unit of intent — adding a feature, fixing a bug, restructuring one thing. Not "files I happened to touch in this directory."

Heuristics for grouping:
- A new feature/page/component → one commit, even if it spans multiple files (e.g., `index.html` + `css/style.css` + `config/home.json` for a home page build = one commit).
- A bug fix → one commit, just the files needed for the fix.
- A refactor or rename → its own commit, separate from feature work.
- Adding a new skill, doc, or config file unrelated to the main work → its own commit.
- Tooling/meta changes (`.claude/`, `.gitignore`, etc.) → its own commit unless it's tightly coupled to a feature.

If everything was actually one cohesive change, one commit is fine. Don't manufacture splits for the sake of looking thorough — but also don't lump unrelated work together because it's faster.

Sketch the grouping out loud (in your reasoning, not necessarily to the user) before staging anything. If it's ambiguous which group a file belongs in, briefly tell the user the plan and let them confirm.

### 3. Pick a branch name
Format: `<type>/<short-kebab-summary>` or `phase-<N>-<slug>` if the work clearly belongs to a numbered phase from `.memory/PLAN.md`.

Examples:
- `phase-0-scaffolding` — Phase 0 work
- `feat/home-page-hero` — building the home page hero
- `fix/nav-active-state` — fixing a bug
- `chore/add-pr-skill` — adding tooling

Short, lowercase, no spaces. If the user is in a project with a phase convention (check `.memory/PLAN.md` and recent commits), use it. Otherwise default to `<type>/<slug>`.

Create the branch and switch to it: `git checkout -b <branch-name>`.

### 4. Stage and commit each group
For each logical group from step 2:

1. `git add <specific files>` — never `git add -A` or `git add .`. Listing files explicitly is the safety net against accidentally including secrets, scratch files, or work-in-progress from another group.
2. Compose a conventional commit message (see format below).
3. Commit with a HEREDOC so multi-line bodies format right:
   ```bash
   git commit -m "$(cat <<'EOF'
   feat(home): add hero section with intro and tagline

   Pulls headline and tagline from config/home.json so a future
   resume-loader phase can swap identity in one place. Mobile-first
   responsive — hero collapses to single column under 640px.
   EOF
   )"
   ```
4. After every commit, run `git status` to confirm the staging area is clean of the group you just shipped.

If a pre-commit hook fails: read the error, fix the underlying issue, re-stage, and create a **new** commit. Do not `--amend` (the original commit didn't actually happen on hook failure, so amending touches the wrong thing) and never `--no-verify`.

### Conventional commit format

```
<type>(<scope>): <imperative summary, lowercase, no period, ≤72 chars>

<optional body — the why, not the what. Wrap at ~72 chars.>
```

**Types** (pick the most specific that fits):
- `feat` — new user-visible functionality
- `fix` — bug fix
- `docs` — documentation only
- `style` — formatting, whitespace, no logic change
- `refactor` — code restructure, no behavior change
- `perf` — performance improvement
- `test` — adding/fixing tests
- `chore` — tooling, build, deps, meta
- `phase-<N>` — when the project uses phase-based commits (this project does — check `CLAUDE.md` for the convention)

**Scope** is optional but helpful. For this project: `home`, `resume`, `projects`, `css`, `config`, `memory`, `skill`, etc.

**Examples:**
- `phase-0: scaffold html, css tokens, and per-page configs`
- `feat(home): add hero section pulling from config/home.json`
- `fix(nav): mark active link with aria-current on every page`
- `chore(skill): add pr skill for end-of-session shipping`

If the project uses a `phase-<N>: <what>` convention (check `CLAUDE.md`), prefer that over `feat:` for phase work. Match the project's existing style — that's why step 1 includes `git log -10`.

### 5. Push the branch
```bash
git push -u origin <branch-name>
```

The `-u` sets upstream so future pushes from this branch are one-word.

### 6. Open the PR

```bash
gh pr create --base master --head <branch-name> --title "<title>" --body "$(cat <<'EOF'
<body>
EOF
)"
```

**Title**: short (≤70 chars), matches the dominant theme of the work. If the branch is one logical thing, mirror the main commit's summary. If it's mixed, pick the umbrella ("Phase 0 scaffolding", "Add pr skill and smoke-test fixes").

**Body**: use this template. Skip sections that don't apply rather than padding them with "N/A".

```markdown
## Summary
- <bullet per logical change, one line each — what shipped, not how>
- <...>

## Why
<1–3 sentences on motivation. Reference the phase, issue, or user request that prompted this. If it's part of a phase from .memory/PLAN.md, name the phase and link the "done when" criteria.>

## Changes
<grouped by area if helpful, e.g.:>
**HTML**
- `index.html` — added hero section, pulled headline from config
- `resume.html` — boilerplate scaffold (Phase 2 will fill in)

**CSS**
- `css/style.css` — added section 6 (PAGE SECTIONS — HOME) with hero styles

**Config**
- `config/home.json` — populated with placeholder identity (George P. Burdell)

## Test plan
- [ ] Open `index.html` in a browser, confirm hero renders and palette swaps with OS theme
- [ ] Resize to ≤375px, confirm responsive layout holds
- [ ] Tab through nav, confirm focus outlines visible and skip link appears
- [ ] (optional) Run `/smoke-test` for the full checklist

## Notes
<anything a reviewer should know — known limitations, follow-ups, decisions deferred. Skip if nothing.>
```

After `gh pr create` returns, capture the PR URL from the output and print it to the user — that's the one thing they actually care about seeing.

### 7. If auto-merge was requested

Only run this branch of the skill if the user passed `merge`, `auto`, or `auto merge`.

1. **Enable auto-merge with squash**:
   ```bash
   gh pr merge --auto --squash --delete-branch
   ```
   `--auto` means GitHub will merge it as soon as required checks pass and approvals are met. If the repo has no required checks/approvals, it merges immediately. `--delete-branch` cleans up the remote branch on merge.

2. **Tell the user what happened**: "Auto-merge enabled. GitHub will merge it as soon as checks pass." If checks already passed (or there are none) and it merged immediately, say so.

3. **Switch back to master and sync**:
   ```bash
   git checkout master
   git pull origin master
   git branch -d <branch-name>   # delete local branch — safe because it was merged
   ```
   If `git branch -d` fails because the merge hasn't actually happened yet (auto-merge is queued, not done), skip the delete and tell the user the local branch is still around for cleanup later. Don't force-delete (`-D`) — that's destructive if something went wrong.

4. **Confirm final state**: `git status` and `git log -1 --oneline` so the user can see they're on a clean master at the merged commit.

## Output to the user

When you're done, print a short summary. The user wants to see:
- The branch name
- The list of commits made (one line each)
- The PR URL
- Whether auto-merge was enabled
- Final state (on `<branch>` / on `master`)

Example:

```
✅ Shipped.

Branch: phase-0-scaffolding
Commits:
  - phase-0: scaffold html, css tokens, and per-page configs
  - chore(skill): add smoke-test skill
  - chore(skill): add pr skill

PR: https://github.com/user/repo/pull/42
Auto-merge: enabled (squash)
Now on: master (up to date)
```

Keep it tight. Don't re-summarize the diff — the PR body already does that.

## Rules
- **Never `git add -A` / `git add .` / `git add *`.** Always list files. The user has been bitten by accidental commits before; this is the safety net.
- **Never `--amend`, `--no-verify`, `--force`, `--force-with-lease`** unless the user explicitly asks. Each of these can quietly destroy work.
- **Never push to `master` directly.** This skill is branch-and-PR. If the user wants to push straight to master, they'll do it themselves.
- **Never auto-merge without the explicit argument.** Default behavior is open the PR and stop. Auto-merge is opt-in for a reason — sometimes the user wants to review the rendered diff on GitHub before merging.
- **Stop and ask if the situation is weird.** Detached HEAD, ongoing rebase/merge, mixed unrelated changes that resist grouping, secrets in the diff, an unusual default branch name — any of these should pause for a human, not be papered over.
- If a hook fails after a commit attempt, fix the root cause, then make a new commit. Never bypass.
- If the user is already on a feature branch (not master) when they run this, ask before doing anything: "You're on `<branch>` — push as-is, or branch off this for a new PR?"
