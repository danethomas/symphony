---
name: build-review
description:
  Phase 2 of the multi-agent build pipeline. Claude reviews the implementation
  diff against the spec and produces a structured review artifact. Run after
  build-implement, before build-finalize.
---

# Build — Phase 2: Review

You are the reviewer, not the implementer. Your job is to produce a sharp,
actionable review that the finalizer can act on directly.

## Process

1. Read the merged spec in `docs/specs/features/`
2. Read `.symphony/build-notes.md` for any deviations the implementer flagged
3. Run `git diff main...HEAD` to inspect all changes
4. Invoke Claude as a Staff Architect + Security reviewer:

```bash
SPEC=$(cat docs/specs/features/<slug>.md)
DIFF=$(git diff main...HEAD)
NOTES=$(cat .symphony/build-notes.md 2>/dev/null || echo "None")

claude --print --dangerously-skip-permissions \
  --system "You are a Staff Architect and Security Engineer reviewing a Rails pull request for Essence.ai. Be direct. Severity: MUST FIX / SHOULD FIX / NICE TO HAVE." \
  -p "Review this implementation against the spec.

Produce a structured review with these sections:
## Spec Alignment
For each spec acceptance criterion: ✅ met / ⚠️ partial / ❌ missing

## Risks / Gaps
Numbered list. Severity: MUST FIX / SHOULD FIX / NICE TO HAVE.

## Security Review
Check each item — raise issues or explicitly clear them:
1. Auth & authz — cross-user data access, missing permission checks
2. Data exposure — endpoints leaking more than needed, secrets in logs
3. Injection — unsanitised input to SQL, shell, or external services
4. Session — hijack, replay, or failure to invalidate
5. Rate limiting — expensive/sensitive ops unprotected
6. Token lifecycle — rotatable, revocable, compromise handling

## Simplify Opportunities
What could be simpler without losing correctness?

## Open Questions
Decisions that need human input before merge.

SPEC:
$SPEC

BUILD NOTES:
$NOTES

DIFF:
$DIFF" > .symphony/build-review.md
```

5. Read `.symphony/build-review.md` and verify it is complete and actionable
6. Do NOT modify application code in this phase
7. Do NOT open a PR

## When Done

- `.symphony/build-review.md` exists and has all five sections populated
- If Claude produced an incomplete review, re-run with a stricter prompt
