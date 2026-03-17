---
name: refine-spec
description:
  Three-pass spec refinement using Claude as Architect and Critic. Run this
  before any implementation work on non-trivial features. Produces a
  battle-tested spec doc and opens a DRAFT PR for human review.
---

# Refine Spec

Run this skill when a ticket is tagged for spec work before implementation.
It uses Claude Code as an independent reviewer — Codex drives, Claude advises.

## Phase 1 — Architect (Draft)

Read the issue and any related docs in `docs/specs/`, `docs/decisions/`, and
`docs/reference/`. Write a complete, opinionated spec at
`docs/specs/features/<slug>.md` with these sections:

- **Goal** — one sentence
- **Why This Exists** — problem being solved
- **Desired Experience** — user-facing behaviour
- **Architecture Direction** — key technical decisions
- **Implementation Plan** — phased, with acceptance criteria per phase
- **Testing Strategy** — what needs tests and at what level
- **Open Questions** — unresolved decisions that need human input
- **Non-Goals** — explicit scope boundaries

## Phase 2 — Claude Critic Review

Once the draft spec exists, invoke Claude as a skeptical Staff Engineer:

```bash
DIFF=$(cat docs/specs/features/<slug>.md)
claude --print --dangerously-skip-permissions \
  --system "You are a skeptical Staff Engineer reviewing a spec for a Rails monolith (Essence.ai). Be direct and opinionated. Do not be polite about gaps." \
  -p "Review this spec and produce numbered objections, gaps, and risks. Then run through this security checklist and raise any issues:
1. Auth & authz — Can a user access another user's data? Missing permission checks?
2. Data exposure — Does any endpoint or tool leak more than it should? Tokens/secrets logged?
3. Injection — Is user input passed unsanitised to SQL, shell, or external services?
4. Session — Can sessions be hijacked, replayed, or not invalidated on logout/revoke?
5. Rate limiting — Are expensive or sensitive operations protected against abuse?
6. Token lifecycle — Are tokens rotatable and revocable? What happens on compromise?
If none apply, state that explicitly.

SPEC:
$DIFF" > .symphony/spec-critique.md
```

Append a `## Refinement Log` section to the spec with:
- `### Critique — Turn 1` — paste the contents of `.symphony/spec-critique.md`

## Phase 3 — Architect (Revise)

Read `.symphony/spec-critique.md`. Revise the spec body addressing every
critique. Append to `## Refinement Log`:
- `### Response — Turn 1` — what changed and why, or explicit rejection with reasoning

## Completion

- Run `bin/local bin/rails docs:check` — fix any missing index links
- Branch: `spec/<issue-slug>`
- Open a **DRAFT** PR titled `spec: <issue-title>`
- PR description must NOT use "Closes #NNN" — the issue must stay open for build phase
- Move the Linear issue to `Human Review`
