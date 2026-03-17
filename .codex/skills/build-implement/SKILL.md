---
name: build-implement
description:
  Phase 1 of the multi-agent build pipeline. Implement the feature from the
  merged spec. Do NOT open a PR — leave the workspace ready for the reviewer.
---

# Build — Phase 1: Implement

You are the implementation agent. Read the issue and the merged spec in
`docs/specs/features/` before touching any code.

## Rules

- Do NOT open a PR in this phase
- Do NOT run `/simplify` yet — that is reserved for Phase 3
- Do NOT commit spec files (`docs/specs/`) — those are already merged
- Implement application code, tests, migrations, and reference doc updates
- Leave the workspace in a coherent, buildable state

## Process

1. Read the merged spec end-to-end
2. Run `bin/local bin/rails db:migrate` if migrations are needed
3. Implement against the spec's acceptance criteria, phase by phase
4. Run `bin/local bin/rails test` — fix any failures before proceeding
5. Run `bin/local bundle exec rubocop -A` — fix any offences
6. If you make key tradeoffs or deviate from the spec, write them to
   `.symphony/build-notes.md` (create the `.symphony/` dir if needed)
7. Commit all changes with a clear message using the `commit` skill

## When Done

- All tests pass
- Rubocop clean
- `.symphony/build-notes.md` exists (even if empty — write "No deviations from spec.")
- Changes committed — do NOT push yet
