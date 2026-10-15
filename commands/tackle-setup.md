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
  "worktreeDir": "../{repo}-worktrees",     // where /tackle-ticket puts worktrees when the user opts in
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
  "codexCmd": "codex",
  "terseMode": true,
  "reviewLoopCap": 4
}
```

For a `github` provider use:

```
"ticketViewCmd": "gh issue view {KEY} --comments",
"ticketSearchCmd": "gh issue list --search \"{QUERY}\""
```

Placeholders `{KEY}`, `{QUERY}`, `{slug}`, `{summary}`, `{repo}` are substituted by `/tackle-ticket` at run time. `codexCmd` is the binary or full command the Codex reviewer runs (`{codexCmd}` in the commands); see step 4. It is substituted as-is, so quote a path that contains spaces (e.g. `\"/Applications/My Tools/codex\"`). Write the file as plain JSON, without the `//` comments shown above.

## 4. Verify

- `useCodex` / `codexCmd`: smoke-test with `rm -f <tmp>/codex-ok.txt; <cmd> exec -s read-only --skip-git-repo-check -o <tmp>/codex-ok.txt "Reply with the single word OK." < /dev/null 2>&1` (keep the `< /dev/null`; without it `codex exec` waits on stdin and hangs) and a 600000 ms Bash timeout. It **passes** only if it exits 0 and `<tmp>/codex-ok.txt`, trimmed, is exactly `OK`. Don't check the console for `OK`: Codex echoes the prompt there, so it always matches.
  - `codex` is on PATH and passes: keep `"codexCmd": "codex"`.
  - `codex` is missing, or fails because it is too old (e.g. `ERROR: ... model is not supported when using Codex with a ChatGPT account`, typical of an app-bundled `codex` that lags the account's default model): read the current version with `npx -y @openai/codex --version` and take the semver from its `codex-cli X.Y.Z` line, since npm may print warnings around it. Smoke-test the **pinned** `npx -y @openai/codex@X.Y.Z` itself, and if it passes, write exactly that command as `codexCmd`.
  - `codex` fails for another reason, such as not logged in, network or rate limit: don't switch binaries. Set `"useCodex": false`, show the error, and tell the user to fix it (e.g. `codex login`) and re-run `/tackle-setup`.
  - Nothing passes: set `"useCodex": false` and tell the user why (the Codex reviewer is skipped; the 3 model reviewers still run).
- Confirm the file is valid JSON.
- Tell the user setup is done and they can now run `/tackle-ticket <KEY>`.

This command is idempotent — re-run any time conventions change.
