---
name: autopilot
description: Runs long, recurring tasks (weekly, monthly, period-close) by turning them into a saved, numbered runbook and executing it step by step, always in the same order, with explicit verification and a log of each run. Use whenever the user describes a routine that repeats ("every week I...", "every month-end...", "that update again"; in Portuguese "toda semana eu...", "todo fechamento...", "de novo aquela atualização", "roda de novo"), asks to automate, standardize or "put on autopilot" ("piloto automático") a multi-step process, asks to resume a routine done before, or mentions a many-step task they keep postponing, even without the word "automate".
---

# Autopilot

Turns a long, repetitive task into a **runbook** saved to a file and executes it step by step, the same way every time. Recurring tasks usually go wrong for three reasons: a step is forgotten, the order drifts from one run to the next, or nobody checks the result. This skill addresses all three: a written runbook, in-order execution, and explicit verification.

Talk to the user in the language they use. Write the runbook in that language too.

## Flow overview

1. Identify the routine and look for an existing runbook
2. If none exists, design the runbook with the user (once)
3. Execute step by step, tracking progress
4. Verify the result against objective criteria
5. Log the run and improve the runbook

---

## 1. Identify the routine and find the runbook

Give the routine a short kebab-case name (e.g. `weekly-model-update`, `monthly-close-client`).

### Where the runbook lives

The runbook has to survive between sessions, so where it is stored depends on the environment. Identify the environment and use **one** storage location, in the priority order below. Step 1 (find) and step 5 (save) use that same location.

| Priority | Environment | How to recognize it | Where to read and write |
|---|---|---|---|
| A | Persistent local folder (Claude Code, or a chat with a computer folder connected) | There is a project directory or connected folder that outlives the session | File `runbooks/<name>.md` at the root of that folder |
| B | claude.ai chat inside a **Project** | A Project documents tool is available (e.g. `Projects` with `project_search` / `project_read` / `project_write`) | Project document at path `runbooks/<name>.md` |
| C | Chat with no persistent storage | None of the above; the chat's filesystem is temporary | The user: ask them to paste the runbook from last time and, at the end, hand over the updated file for them to keep |

**Why this order:** the local folder (A) is the most direct and can be version-controlled. Project documents (B) persist in the cloud and are visible on every device and in every chat of that Project. Manual mode (C) is the last resort, because it depends on the user keeping the file. Saving the same runbook in two places creates diverging copies, so use only the first location available.

**How to look in each location:**
- **A:** list `runbooks/` and read `runbooks/<name>.md`. If the exact name does not exist, list the folder and check whether any runbook matches the routine described, since it may have been saved under another name.
- **B:** search the Project (`project_search`) with keywords from the routine and also check the document list (`project_info`) for paths starting with `runbooks/`. Then read the document you found (`project_read`). When saving, always write to the explicit path `runbooks/<name>.md`, so the runbook does not land in some default folder, and only surface the document to the user when it is created or its steps change, not on every run.
- **Legacy location:** earlier Portuguese versions of this skill saved runbooks under `rotinas/`. If nothing is found under `runbooks/`, also look there; when you find one, keep using it in place rather than duplicating it.
- **C:** ask once: "Do you have the runbook for this routine from last time? If so, paste it here." If not, go to step 2.

If you are in C but the user says the routine recurs, suggest in one line moving the work to a Project (B) or to Claude Code (A), explaining that the runbook will then be found automatically next time.

**Why:** the value of autopilot is repeating the validated process *exactly*. Rebuilding the steps from memory each time reintroduces the variation you are trying to eliminate.

- **Runbook found:** read it in full, show a 2–3 line summary (name, number of steps, date of the last run) and ask only what changed this time (period, new files, exceptions). Then go to step 3.
- **Runbook not found:** go to step 2.

## 2. Design the runbook (first time)

Extract as much as possible from the conversation before asking. Ask at most one round of questions, covering only what is missing:

- **Inputs:** which files, sources or data feed the task, and where they live
- **Steps:** the sequence the user follows today, including the "invisible steps" (renaming a file, checking the reference date, notifying someone)
- **Output:** what is delivered, in what format and to whom
- **Definition of done:** how the user knows it worked (totals that tie, no #REF!, chart updated, etc.)
- **Decision points:** where the process can branch and what to do in each case

Write the runbook with this template and save it as `runbooks/<name>.md` in the location defined in "Where the runbook lives" (local folder, Project document, or a file handed to the user):

```markdown
# Routine: <readable name>
Frequency: <weekly / monthly / on demand>
Last run: <YYYY-MM-DD>

## Inputs
- <file/source> — <where it lives> — <what is expected from it>

## Steps
1. <Imperative verb + object> — Why: <reason> — Done when: <criterion>
2. ...

## Decision points
- If <condition>, then <action>.

## Final verification
- [ ] <objective check 1>
- [ ] <objective check 2>

## Output
<what is delivered, format, destination>

## Log
- <YYYY-MM-DD>: <one-line result; issues found>
```

**Rules for good steps:**
- A step is one action with a verifiable result. If a step has an "and" in the middle, it is probably two steps.
- Every step has a "Why". This lets you adapt intelligently when something changes, instead of blindly following a step that no longer makes sense.
- Every step has a "Done when". Without a criterion there is no way to know whether you can move on.
- Irreversible actions (sending an email, overwriting an original file, publishing, deleting) are marked with ⚠️ and require the user's confirmation at execution time.

Show the runbook to the user and ask for a quick confirmation before running it for the first time.

**Why:** the first design is the investment. A wrong runbook repeated every week multiplies the error, so a minute of review is worth it.

## 3. Execute step by step

- Create a task list with one item per runbook step, so the user can follow progress even if they step away and come back.
- Execute **in order**. Mark a step done only once its "Done when" is satisfied.
- Do not skip steps, even if they look unnecessary this time. If a step genuinely does not apply, record "N/A: <reason>" instead of silently omitting it.
- On ⚠️ steps, stop, show exactly what will be done and wait for a "yes".
- Work on copies of the input files, never the originals, unless the runbook explicitly says otherwise.

**When something goes off-script** (missing file, changed format, number out of range):
1. Stop at the step where the problem showed up, with no silent workaround.
2. Describe the problem in one sentence and propose a fix.
3. If the user is not present, apply the most conservative fix, record it prominently in the final report and continue, unless the step is ⚠️.

**Why:** silent deviation is what makes automated routines dangerous. The user needs to trust that "it worked" means the runbook was followed.

## 4. Verify

Actually run every item in "Final verification": open the file, recompute the total, check the date, compare with the previous run where it makes sense (large swings deserve a comment). Do not declare the routine done just because "the steps ran".

**Why:** finishing the steps does not guarantee a correct result. Verification is what turns an execution into a reliable deliverable.

## 5. Log and improve

- Update "Last run" and add a line to the runbook's "Log", saving to the same location it was read from:
  - **A (local folder):** edit `runbooks/<name>.md`.
  - **B (Project):** read the current document, apply the changes and write the full content back to the same path (`project_write`), since there is no partial edit. This avoids wiping changes another session may have made.
  - **C (manual):** hand over the updated `<name>.md` file and remind the user, in one line, to replace the old version.
- If any step needed adjusting, a workaround, or turned out to be unnecessary, propose the runbook change to the user (show before and after) and apply it if they approve.
- Keep the log lean: after about 10 lines, summarize the oldest into one line.

**Why:** a runbook that does not learn from its runs goes stale and brings the rework back.

## Final report to the user

Keep it short and concrete:

```
✅ <Routine> done — <date>
Delivered: <file/output>
Verification: <n>/<n> checks passed
Deviations: <none | short list of what was done>
Runbook updated: <yes/no — what changed>
```

## Trigger examples

- "Every Monday I update the model with the closing prices, redo the multiples and send the summary. Do it for me."
- "Bora rodar de novo aquele fechamento do mês."
- "I have a 12-step checklist to reconcile cash and I always forget something. Standardize it."
