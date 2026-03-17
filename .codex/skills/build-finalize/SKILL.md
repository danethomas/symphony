---
name: build-finalize
description:
  Phase 3 of the multi-agent build pipeline. Read the review artifact, apply
  fixes, run /simplify, then open the PR with a structured description. This
  is the only phase that pushes and opens a PR.
---

# Build — Phase 3: Finalize

Read `.symphony/build-review.md` carefully before touching anything.

Refer to this repo's `WORKFLOW.md` for the correct commands to run tests,
linters, and any pre-PR checks — do not assume a specific stack.

## Process

1. **Address the review**
   - Apply every MUST FIX item — no exceptions
   - Apply SHOULD FIX items unless there is a clear, documented reason not to
   - NICE TO HAVE items are optional — note which you skipped and why

2. **Run CI** — use the test and lint commands from `WORKFLOW.md`
   Fix any failures before continuing.

3. **Run simplify**
   ```bash
   claude --print --dangerously-skip-permissions -p "/simplify" > .symphony/simplify-suggestions.md
   ```
   This runs Claude Code's built-in `/simplify` command — a structured pass
   that identifies and applies safe simplifications. Incorporate any changes
   it makes. Note what you kept vs reverted.

4. **Commit** using the `commit` skill

5. **Push** using the `push` skill

6. **Open the PR** using `gh pr create` with this EXACT template — do not omit sections:

```bash
gh pr create \
  --title "<type>: <short description>" \
  --body "## Implemented from Spec
<bullet list of what was built, referencing spec acceptance criteria>

## Reviewer Findings
<!-- Copy HIGH and MUST FIX items from .symphony/build-review.md verbatim -->

## Revisions Made
<!-- For each finding: what changed, or 'Not addressed — <reason>' -->

## Simplify Pass
<!-- What was simplified, or 'No significant concerns raised' -->

## Open Questions
<!-- Copy Open Questions from build-review.md with your answers where possible -->

Linear: <issue URL>" \
  --base main
```

Do not open the PR until all sections are filled in.

7. Move the Linear issue to `Human Review`

## Checklist Before Done

- [ ] All MUST FIX review items addressed
- [ ] Tests and linter passing
- [ ] Simplify pass complete
- [ ] PR open with all template sections populated
- [ ] Linear ticket in Human Review
