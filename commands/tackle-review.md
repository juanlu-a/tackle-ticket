---
description: Multi-model review loop on the current diff — no ticket system needed. Fan out, dedupe, fix, re-review until clean.
argument-hint: [optional: base branch, PR number, or a sentence describing what the change should do]
---

Run the review loop on the work in progress: **$ARGUMENTS**

This is `/tackle-ticket` steps 5 and 6 on their own, for when there is no ticket to read — a
branch you already wrote, someone else's PR, or a change that never had a ticket.

Load config from `.claude/tackle-ticket.json` if it exists, for `reviewModels`, `reviewLoopCap`,
`useCodex`, `terseMode`, `baseBranch`, `lintCmd` and `typecheckCmd`. **If it does not exist, do not
stop** — unlike `/tackle-ticket`, this command works without config. Fall back to: `reviewModels`
`["opus", "sonnet", "haiku"]`, `reviewLoopCap` 4, `useCodex` true, `terseMode` true, `baseBranch`
the repo's default branch, and detect lint/typecheck from `package.json` scripts (or skip them and
say so).

## 1. Work out what to review, and against what

The diff:

- A number in the argument (`123`, `#123`) → that PR: `gh pr diff <n>`, and note its head branch.
- A branch name → `git diff <that branch>...HEAD`.
- Nothing → `git diff <baseBranch>...HEAD`. If that is empty, fall back to the uncommitted working
  tree (`git diff HEAD`). If both are empty, say so and stop.

The intent — this replaces the ticket, and reviewers are much weaker without it:

1. The sentence in the argument, if the user wrote one.
2. Otherwise the PR body (`gh pr view <n> --json title,body`) when reviewing a PR.
3. Otherwise the commit messages on the branch (`git log <base>..HEAD`). These often carry the
   *why*, which is exactly what a reviewer needs.
4. If all of that is thin — a one-line commit and no PR body — **ask the user what this change is
   supposed to do before spawning anyone.** One question. A reviewer with no intent finds style
   nits and misses the bug.

Also collect the conventions: the repo's `CLAUDE.md` or `AGENTS.md`, and any skill the repo loads
for this kind of work. Pass the **relevant excerpt**, not the whole file.

## 2. Review loop (until clean)

Repeat until a round produces **no `critical` and no `major` findings**, or `reviewLoopCap` rounds
are reached. If the cap is hit, list what remains instead of pretending it is clean.

**2a. Fan out in ONE turn (parallel):**

- For each model in `reviewModels`, spawn a `ticket-reviewer` subagent via the Agent tool with
  `subagent_type: "ticket-reviewer"` and `model:` set to that model. (Do NOT use
  `subagent_type: "fork"` — it ignores the model override.)
- Where that agent expects the ticket and acceptance criteria, give it the intent from step 1 and
  say plainly that there is no ticket: judge the change against its stated intent and the repo's
  conventions, not against invented requirements.
- If `useCodex` is true, also run the Codex reviewer as a Bash call in the same turn:

  ```bash
  codex exec -s read-only "Review this diff. Report findings only — do NOT edit files. Terse: one finding per line as [severity] file:line — problem. Fix: change. Severity = critical|major|minor|nit; output NO FINDINGS if clean.

  INTENT: <what the change should do>
  CONVENTIONS: <excerpt>
  DIFF:
  <the diff>"
  ```

  If `codex` is not on PATH, skip it silently.

All reviewers are **report-only** and output in terse-mode style.

**2b. Synthesize:**

Dedupe (same file:line + same issue) and rank by severity. List the deduped findings to the user,
terse. When reviewers disagree, say so rather than silently picking one — a split verdict is
information.

**2c. Apply:**

Fix every `critical` and `major`. Apply `minor`/`nit` when cheap and safe; otherwise note them.
Update or add tests for anything the fixes change. Then loop back to 2a, re-reviewing **only the
files you just touched**.

**Reviewing someone else's PR is report-only.** If the diff came from a PR whose head branch is not
checked out, do not fix anything: report, and offer to post the findings as PR comments.

## 3. Lint and typecheck

Run `lintCmd` and `typecheckCmd` and fix what they flag. Skip with a note if the repo has neither.

## 4. Stop

Report what was found, what was fixed and what remains. **Do not commit, push, open a PR or merge
anything** — this command reviews. If the user wants the fixes committed, they will say so.
