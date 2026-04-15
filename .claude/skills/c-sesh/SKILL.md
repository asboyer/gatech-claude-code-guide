# Skill: close-session

## Usage
```
/close-session <summary>
/close-session Finished the home page hero section and nav links
/close-session
```
If no summary is provided, ask the user what was accomplished before proceeding.

## Inputs
- **Summary** (required, will prompt if missing): what was accomplished this session
- **Next steps** (optional): what should happen next session

## Steps

1. **Find the open session.** List `.memory/sessions/` and find the most recent file with `Status: 🟢 Open`.
   - If multiple open sessions exist, list them and ask the user which one to close.
   - If no open session exists, print `"No open session found."` and stop.

2. **Update the session file.** In the open session file:
   - Change `Status: 🟢 Open` → `Status: ✅ Closed`
   - Add `Closed: <current timestamp>` on the line below `Started:`
   - Fill in `## Work Log` with a summary of what was done
   - Fill in `## Open Questions` with anything unresolved, or `None`

3. **Update `.memory/DECISIONS.md`.** If any decisions were made during the session (user mentioned them, or they're logged in the session's `## Decisions Made`), append each one:
   ```markdown
   ## D-<next number>: <title>
   **Session:** <session filename>
   **Context:** <why this came up>
   **Decision:** <what was chosen>
   **Reasoning:** <why>
   ```
   Number sequentially (D-001, D-002, etc.) based on what's already in the file.

4. **Update `.memory/STATUS.md`.** Overwrite with current state:
   - Current phase — only advance if every "done when" bullet from `PLAN.md` is met. If unsure, keep the current phase and note what's remaining.
   - Today's date
   - One-line summary of what changed

   Format:
   ```markdown
   # Project Status

   Stack: <keep existing>
   Hosting: <keep existing>
   Build command: <keep existing>
   Test command: <keep existing>
   Lint command: <keep existing>
   Git: <keep existing>
   Initialized: <keep existing>
   Last updated: <today's date>
   Current phase: <phase number — name>
   Last session: <one-line summary>
   ```

5. **Update `.memory/LAST_SESSION.md`.** Overwrite the entire file:
   ```markdown
   # Last Session

   **Date:** <today>
   **Session file:** <path to the session file>
   **Phase:** <phase number and name>

   ## What Was Done
   <bullet list of accomplishments>

   ## What's Next
   <user-provided next steps, or inferred from PLAN.md>

   ## Open Questions
   <anything unresolved, or "None">
   ```

6. **Output a closing summary.** Print to chat:
   - What was accomplished
   - Whether the phase is complete or still in progress (and what remains if in progress)
   - What's queued for next session
   - Any open questions or blockers

## Rules
- Do NOT do any project work. This skill only handles bookkeeping.
- If the user didn't provide a summary, ask for one. Do not invent what was done.
- Be conservative about phase advancement. Only move forward if every "done when" item is satisfied. When in doubt, keep the current phase.
- Always update all four files: the session file, `DECISIONS.md`, `STATUS.md`, and `LAST_SESSION.md`.
