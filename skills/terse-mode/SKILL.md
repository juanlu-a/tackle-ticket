---
name: terse-mode
description: Token-saving output style for review and reporting. Drop filler, keep substance, use fragments — but never alter code, commands, paths, or error text. Applied automatically by tackle-ticket to the review subagents, the Codex reviewer, and the final synthesis/report. Also usable on demand for any terse output.
---

# Terse Mode

Write to be read fast and cheap. Compress the *prose*, never the *substance*.

Adapted from [caveman](https://github.com/juliusbrussee/caveman) by Julius Brussee (MIT).

## Rules

- Drop filler: no preamble, no "I'll now…", no restating the request, no summaries of what you're about to do.
- Fragments over sentences. Bullets over paragraphs. One idea per line.
- Cut adjectives, hedges, and pleasantries. State the finding, not your feelings about it.
- Lead with the conclusion. Reasoning only when it changes the action.
- Merge related points; don't repeat context the reader already has.

## Never touch (byte-for-byte exact)

- Code, diffs, and snippets.
- Shell commands and flags.
- File paths and `file:line` references.
- Error messages, stack traces, log lines.
- Identifiers (function/var/type names), config keys, URLs.

Accuracy on these beats brevity. If unsure whether shortening changes meaning, keep it.

## Examples

Normal: "You should probably wrap this object in `useMemo`, since a new reference is created on every render which can cause unnecessary re-renders downstream."
Terse: "New ref each render → downstream re-renders. Wrap in `useMemo`."

Normal: "I reviewed the diff and I think there might be an issue where the null case isn't handled in the parser function."
Terse: "`parse()` unhandled null case — crashes on empty input."

## Finding format (for reviewers)

```
[SEVERITY] file:line — problem. Fix: <concrete change>.
```

`SEVERITY` ∈ `critical | major | minor | nit`. One finding per line. No intro, no outro.
