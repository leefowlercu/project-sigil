# PRD

This directory stores product requirement documents for the `sigil-web`
subproject.

- PRDs define expected external behavior and acceptance criteria.
- Organize PRDs by stable subsystem, route surface, or mechanism ownership.
- Update PRD acceptance scenarios before or alongside `sigil-web` implementation
  changes.
- Use scenario IDs in the form `SCN-xxxx` for PRD acceptance scenarios.
- Use PRD filenames in the form `PRD-<4-digit>-<slug>-specification.md`.
- Keep PRD scenario IDs and titles aligned with
  [`docs/sigil-web/PRD/MATRIX.md`](MATRIX.md).

## Management Model

- Keep PRDs behavior-centric and acceptance-backed.
- Prefer one owning PRD per durable concern boundary.
- Reuse scenario titles when behavior is unchanged so traceability stays
  mechanical and reviewable.
- When a PRD is replaced or split, update this README and the matrix in the same
  change.

## Lifecycle and Verification

- Keep PRD acceptance scenario titles globally unique within `sigil-web`.
- If multiple PRDs need to mention the same behavior, keep the acceptance
  scenario in one owning PRD and reference that owner from the others.
- Run `./scripts/verify-specs --subproject sigil-web` after structural PRD,
  matrix, or acceptance-title edits.

## Numbering Blocks

- `PRD-0100` to `PRD-0199`: shell and route surface
- `PRD-0200` to `PRD-0299`: workspace and detail surfaces
- `PRD-0300` to `PRD-0399`: live app-server session and data flow

## Current PRDs

- [`PRD-0100-sigil-web-application-shell-and-route-surface-specification.md`](PRD-0100-sigil-web-application-shell-and-route-surface-specification.md):
  Shared shell and route-surface contract
- [`PRD-0200-sigil-web-agent-workspace-selection-and-fleet-management-specification.md`](PRD-0200-sigil-web-agent-workspace-selection-and-fleet-management-specification.md):
  Agent workspace selection and fleet-management contract
- [`PRD-0210-sigil-web-run-detail-workspace-specification.md`](PRD-0210-sigil-web-run-detail-workspace-specification.md):
  Root and standalone run-detail workspace contract
- [`PRD-0300-sigil-web-live-app-server-session-and-run-subscription-specification.md`](PRD-0300-sigil-web-live-app-server-session-and-run-subscription-specification.md):
  Live session and run-subscription contract

## Next PRD

Create the next record within the correct subsystem block using the first open
number in that block.
