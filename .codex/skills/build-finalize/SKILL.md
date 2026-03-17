---
name: build-finalize
description:
  Phase 3 of the multi-agent build pipeline. Read the review artifact, apply
  fixes, run /simplify, then open the PR with a structured description. This
  is the only phase that pushes and opens a PR.
---

# Build — Phase 3: Finalize

Read `.symphony/build-review.md` carefully before touching anything.

## Process

1. **Address the review**
   - Apply every MUST FIX item — no exceptions
   - Apply SHOULD FIX items unless there is a clear, documented reason not to
   - NICE TO HAVE items are optional — note which you skipped and why

2. **Run CI**
   ```bash
   bin/local bin/rails test
   bin/local bundle exec rubocop -A
   ```
   Fix any failures before continuing.

3. **Run simplify**
   ```bash
   claude --print --dangerously-skip-permissions \
     --system "You are a senior Rails engineer. Identify code that can be simplified without changing behaviour. Be conservative — only suggest changes you are confident are safe." \
     -p "$(git diff main...HEAD)" > .symphony/simplify-suggestions.md
   ```
   Incorporate safe simplifications. Note what you applied vs skipped.

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
- [ ] `bin/ci` passing
- [ ] Simplify pass complete
- [ ] PR open with all template sections populated
- [ ] Linear ticket in Human Review
