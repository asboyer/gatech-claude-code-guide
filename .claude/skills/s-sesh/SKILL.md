# Skill: start-session

## Usage
```
/start-session <goal>
/start-session Build out the home page
/start-session Phase 2 — add print styles to resume
```

## Inputs
- **Goal** (required): what this session is for
- **Phase** (optional): if not provided, infer from the goal + current phase in `STATUS.md`

## Steps

1. **Read current state.** Open and read:
   - `.memory/STATUS.md` — current phase and last known state
   - `.memory/LAST_SESSION.md` — what happened last time, what was queued as "next"
   - `.memory/DECISIONS.md` — skim for recent decisions that affect this session
   - `.memory/PLAN.md` — read the phase relevant to this session's goal, especially scope and "done when" criteria

2. **Check for parallel sessions.** List all files in `.memory/sessions/`. Any file that does NOT have `Status: ✅ Closed` is an open session. If open sessions exist:
   - Print: `⚠️ Open sessions detected:` and list them with their goals and phases
   - Note which files/phases those sessions touch so the user can avoid conflicts
   - Do NOT block — parallel sessions are allowed

3. **Create the session file.** Write to `.memory/sessions/<timestamp>-<slug>.md` where:
   - `<timestamp>` = `YYYY-MM-DD-HHMM` (current time)
   - `<slug>` = short kebab-case from the goal (e.g., `home-page`, `resume-print-styles`)

   Contents:
   ```markdown
   # Session: <goal>

   Phase: <phase number and name>
   Started: <timestamp>
   Status: 🟢 Open

   ## Goal
   <the user's stated goal>

   ## Scope
   <bullet list of deliverables, pulled from PLAN.md's phase definition>

   ## Done When
   <the "done when" criteria from PLAN.md for this phase>

   ## Work Log
   <!-- Append entries as work progresses -->
   <!-- - <time>: <what was done> -->

   ## Decisions Made
   <!-- Decisions made this session — also append to .memory/DECISIONS.md at close -->

   ## Open Questions
   <!-- Anything unresolved that carries forward -->
   ```

4. **Output a session briefing.** Print to chat:
   - Current phase and session goal
   - Any open parallel sessions and what they're touching
   - What happened last session (from `LAST_SESSION.md`) if relevant
   - The "done when" criteria for this phase
   - `"Session is open. Let's get started."`

## Rules
- Do NOT start doing project work. This skill only reads state, creates the session file, and briefs the user.
- Do NOT modify `STATUS.md`, `DECISIONS.md`, or `LAST_SESSION.md`. That happens at close.
- If the goal doesn't map to any phase in `PLAN.md`, flag it but still create the session.
