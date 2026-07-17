# Dogfooding log

Running notes from real use, to drive the polish pass before promoting. Delete before/ignore for the public release, or keep as a changelog seed.

Log each real run: what worked, what felt clunky, anything that broke. Rough is fine — signal over polish.

## Rubric for the polish pass (what "stylish + useful + perfect" means here)

- [ ] **Output style** — review findings are scannable (grouped by severity, terse, no wall of text); progress is legible during the 4-reviewer fan-out.
- [ ] **Correctness** — all 4 reviewers actually fire; model overrides land on the right models; Codex read-only never edits; loop terminates and respects the cap.
- [ ] **Token cost** — terse-mode meaningfully cuts review output; context stays diff-scoped.
- [ ] **Config UX** — `/tackle-setup` detection is accurate; missing-config and missing-Codex paths degrade gracefully.
- [ ] **Docs/demo** — README has a real before/after + a short recording/GIF for the post.
- [ ] **Robustness** — sensible behavior on: no ticket found, dirty working tree, lint/tsc failures, empty diff.

## Run log

<!-- template
### <date> — CSD-XXXX (<one-line ticket>)
- Went well:
- Clunky:
- Broke:
- Idea:
-->
