# tackle-ticket

A Claude Code plugin that takes a ticket from key to PR:

**fetch ticket → plan + ask → branch → code + tests → multi-model review loop → open PR.**

After coding, it fans out **four read-only reviewers in parallel** — Opus, Sonnet, and Haiku subagents plus [Codex](https://github.com/openai/codex) (`codex exec`, if installed) — synthesizes their findings, applies fixes, and **loops until a review round is clean** (with a safety cap). Reviewers report only; the main agent applies corrections, so there are no conflicting parallel edits.

It's **generic and config-driven**: run `/tackle-setup` once per repo to detect your ticket system (Jira or GitHub Issues), base branch, branch-naming, test/lint/typecheck commands, and PR conventions into `.claude/tackle-ticket.json`. The plugin stays project-agnostic; the config makes it yours.

## Commands

| Command | What it does |
| --- | --- |
| `/tackle-setup` | Detect this repo's conventions and write `.claude/tackle-ticket.json`. Run once per repo (idempotent). |
| `/tackle-ticket <KEY \| description>` | Run the full workflow for a ticket. |

## Install

```
/plugin marketplace add <path-or-git-url-to-this-repo>
/plugin install tackle-ticket@tackle-ticket
```

Then, in each repo you want to use it in:

```
/tackle-setup
```

### Requirements

- A ticket CLI: [`jira`](https://github.com/ankitpokhrel/jira-cli) (Jira) or [`gh`](https://cli.github.com/) (GitHub Issues) + `gh` for PRs.
- Optional: [Codex CLI](https://github.com/openai/codex) for the 4th reviewer. If absent, the three model reviewers still run.

## Token optimization (terse mode)

The token-heavy review phase runs in a bundled **terse-mode** — reviewers and the final synthesis drop filler and use fragments while keeping code, commands, paths, and errors byte-for-byte exact. Context given to reviewers is scoped to the diff + a conventions excerpt (not the whole repo), and re-reviews are scoped to changed files.

Terse-mode is adapted from [**caveman**](https://github.com/juliusbrussee/caveman) by Julius Brussee (MIT). This plugin vendors a lightweight variant so no external install is required; for the full caveman feature set, install it separately.

## License

MIT. See [LICENSE](./LICENSE). Terse-mode is derived from caveman (MIT), attributed above.
