# PRD

This directory stores product requirement documents for the `sigil-web` subproject.

- PRDs define expected external behavior and acceptance criteria.
- Organize PRDs by stable operator workflow or concern boundary, not by delivery
  chronology.
- Prefer one normative owner for a behavior; other PRDs should reference that
  owner instead of restating the contract.
- Update PRD acceptance scenarios before or alongside submodule implementation changes.
- Use scenario IDs in the form `SCN-xxxx` for PRD acceptance scenarios.
- Use PRD filenames in the form `PRD-<4-digit>-<slug>-specification.md`.
- Keep PRD scenario IDs and titles aligned with `docs/sigil-web/PRD/MATRIX.md` mappings.
- Keep routed UI scenarios aligned with
  [`sigil-web/design/design-manifest.toml`](../../../sigil-web/design/design-manifest.toml).

## Management Model

- Keep PRDs behavior-centric and acceptance-backed.
- Keep each PRD focused on one durable concern boundary:
  - session and transport state
  - agent hub and fleet selection
  - run inspection workspace
  - live orchestration and recovery
  - agent-scoped run control and authoring
- Prefer adding a new PRD over stretching an existing record across multiple
  workflow boundaries.

## Lifecycle and Verification

- Keep PRD acceptance scenario titles globally unique within `sigil-web`.
- Keep routed UI state names stable across PRDs, the acceptance feature file,
  and the design manifest so route-state traceability stays mechanical.
- Run `./scripts/verify-specs --subproject sigil-web` after structural PRD,
  matrix, feature-title, or design-manifest edits.

## Numbering Blocks

- `PRD-0100` to `PRD-0199`: session, transport, and compatibility
- `PRD-0200` to `PRD-0299`: agent hub and fleet-selection workflows
- `PRD-0300` to `PRD-0399`: run inspection and detail workspace behavior
- `PRD-0400` to `PRD-0499`: live orchestration and connection recovery
- `PRD-0500` to `PRD-0599`: agent-scoped run control and authoring

## Current PRDs

### Session, Transport, and Compatibility

- [PRD-0100 Sigil-Web Session and Connection State Specification](PRD-0100-sigil-web-session-and-connection-state-specification.md):
  WebSocket handshake, compatibility, and connection-state UX contract

### Agent Hub and Fleet Selection

- [PRD-0200 Sigil-Web Agent Fleet Hub and Selection Specification](PRD-0200-sigil-web-agent-fleet-hub-and-selection-specification.md):
  `/agents` command hub, fleet selection, deep-linking, and fleet empty-state contract

### Run Inspection and Detail Workspace

- [PRD-0300 Sigil-Web Run Detail Workspace Specification](PRD-0300-sigil-web-run-detail-workspace-specification.md):
  Summary, tree, timeline, step, and artifact inspection contract

### Live Orchestration and Connection Recovery

- [PRD-0400 Sigil-Web Live Orchestration and Connection Recovery Specification](PRD-0400-sigil-web-live-orchestration-and-connection-recovery-specification.md):
  Live subscription, reconnect, and terminal transition contract

### Agent-Scoped Run Control and Authoring

- [PRD-0500 Sigil-Web Agent Run Control and Authoring Specification](PRD-0500-sigil-web-agent-run-control-and-authoring-specification.md):
  Selected-agent new-run dialog plus run-stop control contract

## Next PRD

Create the next record within the correct subsystem block using the first open
number in that block.

Examples:

- `docs/sigil-web/PRD/PRD-0150-<slug>-specification.md` for new session or compatibility behavior
- `docs/sigil-web/PRD/PRD-0350-<slug>-specification.md` for new run-detail workspace behavior
- `docs/sigil-web/PRD/PRD-0550-<slug>-specification.md` for new remote-control behavior
