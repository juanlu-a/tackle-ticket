---
description: Configure tackle-ticket for THIS repo — detect ticket system + conventions and write .claude/tackle-ticket.json.
argument-hint: (no args; run once per repo, re-run to update)
---

Turn the generic `tackle-ticket` plugin into a project-configured one by inspecting this repository and writing `.claude/tackle-ticket.json`.

## 1. Detect

Gather signals (read-only):

- **Conventions doc:** look for `CLAUDE.md`, then `README.md`, `CONTRIBUTING.md`. Read whichever exists for branch/PR/commit/test rules.
- **Ticket provider:**
  - `command -v jira` present and a Jira key prefix visible in git history (`git log --oneline -30`) → `jira`.
  - else if `command -v gh` and a GitHub remote (`git remote -v`) → `github`.
- **Base branch:** `git symbolic-ref refs/remotes/origin/HEAD` or presence of `staging` vs `main`/`master`.
- **Ticket key prefix:** infer from recent branch/commit names (e.g. `CSD-`).
- **Commands:** read `package.json` scripts (or Makefile/pyproject) for `test`, `lint`, `typecheck`/`tsc`, and the package manager (`pnpm`/`npm`/`yarn` via lockfile).
- **Commit/PR conventions:** conventional commits? co-author line? PR title format? PR footer? — from the conventions doc.

## 2. Propose

Show the proposed config as a table and ask the user to confirm or adjust anything ambiguous (base branch, provider, commands, PR format). Do not guess silently on things that change behavior.

## 3. Write `.claude/tackle-ticket.json`

Shape:

```jsonc
{
  "ticketProvider": "jira",                 // "jira" | "github"
  "ticketViewCmd": "jira issue view {KEY} --plain --comments 5",
  "ticketSearchCmd": "jira issue list --plain -q \"{QUERY}\"",
  "baseBranch": "staging",
  "branchNaming": "{KEY}-{slug}",
  "quickfixNaming": "quickfix-{slug}",
  "testCmds": ["pnpm test"],
  "lintCmd": "pnpm lint",
  "typecheckCmd": "pnpm tsc",
  "prTitleFormat": "[{KEY}] {summary}",
  "prBase": "staging",
  "commitConvention": "conventional-no-scope",
  "coAuthorLine": "Co-Authored-By: Claude <noreply@anthropic.com>",
  "prFooter": "🤖 Generated with [Claude Code](https://claude.com/claude-code)",
  "conventionsDoc": "CLAUDE.md",
  "reviewModels": ["opus", "sonnet", "haiku"],
  "useCodex": true,
  "terseMode": true,
  "reviewLoopCap": 4
}
```

For a `github` provider use:

```
"ticketViewCmd": "gh issue view {KEY} --comments",
"ticketSearchCmd": "gh issue list --search \"{QUERY}\""
```

Placeholders `{KEY}`, `{QUERY}`, `{slug}`, `{summary}` are substituted by `/tackle-ticket` at run time.

## 4. Verify

- `useCodex`: check `command -v codex`. If absent, set `"useCodex": false` and note it (Codex reviewer is skipped, the 3 model reviewers still run).
- Confirm the file is valid JSON.
- Tell the user setup is done and they can now run `/tackle-ticket <KEY>`.

This command is idempotent — re-run any time conventions change.
