---
description: Tackle a ticket end-to-end — fetch → plan → branch → code + tests → multi-model review loop → PR.
argument-hint: [TICKET-KEY or a description of the work]
---

Take this ticket from start to PR: **$ARGUMENTS**

Load config from `.claude/tackle-ticket.json`. If it does not exist, tell the user to run `/tackle-setup` first (offer to auto-detect conventions for this one run as a fallback). All `{KEY}`, `{slug}`, `{summary}`, `{QUERY}` below come from that config's templates.

Follow these steps in order.

## 1. Understand the ticket

- If the argument is a ticket key, fetch it with the config's `ticketViewCmd` (substitute `{KEY}`). Read the description + acceptance criteria before planning.
- If it's a description, find the ticket with `ticketSearchCmd`; confirm the match with the user if ambiguous.
- If the ticket CLI errors on auth, stop and tell the user how to authenticate — don't work around it.

## 2. Plan and ask BEFORE coding

- Produce a short plan and use plan mode (`ExitPlanMode`) so the user approves it.
- Ask up front about unknowns, edge cases, scope, and design choices. Don't guess on anything that changes what you build.
- Do not write the solution until the plan is approved.

## 3. Branch from the base branch

```bash
git checkout <baseBranch> && git pull
git checkout -b <branchNaming>    # e.g. CSD-1234-short-slug ; quickfix-<slug> if no ticket
```

Use the config's `baseBranch` and `branchNaming`. Respect any repo-specific exception in the conventions doc (e.g. mobile branch cascades).

## 4. Implement with tests

- Implement the change. Add tests covering the behavior — not just the happy path — using the repo's test framework.
- Honor the conventions doc (e.g. one migration per PR).

## 5. Multi-model review loop (until clean)

Repeat the following round until a round produces **no `critical` and no `major` findings**, or `reviewLoopCap` rounds are reached (log what remains if the cap is hit).

**5a. Build the review context (token-scoped):**
- `git diff <baseBranch>...HEAD` (round 1) or a diff of only the files you changed since last round.
- The ticket text + acceptance criteria.
- The relevant excerpt of the conventions doc (not the whole repo).

**5b. Fan out reviewers in ONE turn (parallel):**
- For each model in `reviewModels` (default `opus`, `sonnet`, `haiku`), spawn a `ticket-reviewer` subagent via the Agent tool with `subagent_type: "ticket-reviewer"` and `model:` set to that model. Pass the review context in the prompt. (Do NOT use `subagent_type: "fork"` — it ignores the model override.)
- If `useCodex` is true, also run the Codex reviewer as a Bash call in the same turn:
  ```bash
  codex exec -s read-only "Review this diff for a ticket. Report findings only — do NOT edit files. Terse: one finding per line as [severity] file:line — problem. Fix: change. Severity = critical|major|minor|nit; output NO FINDINGS if clean.

  TICKET: <key + summary + acceptance criteria>
  CONVENTIONS: <excerpt>
  DIFF:
  $(git diff <baseBranch>...HEAD)"
  ```
  If `codex` is not on PATH, skip it silently (the model reviewers still run).

All reviewers are **report-only** and output in terse-mode style.

**5c. Synthesize:**
- Collect the 4 reviewers' findings. Dedupe (same file:line + same issue). Rank by severity.
- Briefly list the deduped findings to the user (terse).

**5d. Apply corrections:**
- Fix every `critical` and `major` finding. Apply `minor`/`nit` when cheap and safe; otherwise note them.
- Update/add tests for anything the fixes change.
- Then loop back to 5a, re-reviewing only the files you just touched.

## 6. Lint + typecheck

Run the config's `lintCmd` and `typecheckCmd`. Fix anything they flag before opening the PR.

## 7. Open the PR

Only after the above pass. Commit with the repo's convention (e.g. conventional commits without scope), ending the message with `coAuthorLine`. Then:

```bash
gh pr create --base <prBase> --title "<prTitleFormat>" --body "...<prFooter>"
```

## 8. Local test steps

Give brief, exact steps to verify the change locally (commands to run, route/URL to hit, what to look for). Concise.

---

**Token note:** the review phase (step 5) runs in terse-mode automatically via the bundled `terse-mode` skill and the `ticket-reviewer` agent's output contract. Keep your own synthesis and reporting terse too.
