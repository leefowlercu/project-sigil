# PRD

This directory stores product requirement documents for the `sigil-web`
subproject.

- PRDs define expected external behavior and acceptance criteria.
- Organize PRDs by route family, not by delivery chronology.
- Prefer one normative owner for a behavior; other PRDs should reference that
  owner instead of restating the contract.
- Update PRD acceptance scenarios before or alongside submodule
  implementation changes.
- Use scenario IDs in the form `SCN-xxxx` for PRD acceptance scenarios.
- Use PRD filenames in the form `PRD-<4-digit>-<slug>-specification.md`.
- Keep PRD scenario IDs and titles aligned with
  [`docs/sigil-web/PRD/MATRIX.md`](MATRIX.md).
- Keep routed UI scenarios aligned with
  [`sigil-web/verification/scenarios/manifest.toml`](../../../sigil-web/verification/scenarios/manifest.toml).
- Keep scenarios that carry Paper evidence aligned with
  [`sigil-web/verification/design/manifest.toml`](../../../sigil-web/verification/design/manifest.toml).

## Management Model

- Keep PRDs behavior-centric and acceptance-backed.
- Keep each PRD focused on one route family:
  - root and index route
  - `/agents`
  - `/runs/$runId`
- Prefer adding a new PRD over stretching an existing record across multiple
  route families.
- When a PRD is replaced or retired, update this README, the matrix, the
  acceptance file, the scenario manifest, any matching design-manifest entries,
  and affected ADR backlinks in the same change.

## Lifecycle and Verification

- Keep PRD acceptance scenario titles globally unique within `sigil-web`.
- Keep routed UI state names stable across PRDs, the acceptance feature file,
  the scenario manifest, and the design manifest so route-state traceability
  stays mechanical.
- When a PRD is split, replaced, or retired, update the legacy migration map
  and affected ADR backlinks in the same change.
- Run `./scripts/verify-specs --subproject sigil-web` after structural PRD,
  matrix, feature-title, scenario-manifest, or design-manifest edits.

## Numbering Blocks

- `PRD-0100` to `PRD-0199`: root and index route behavior
- `PRD-0200` to `PRD-0299`: `/agents` route behavior
- `PRD-0300` to `PRD-0399`: `/runs/$runId` route behavior

## Current PRDs

### Root and Index Route

- [PRD-0100 Sigil-Web Root and Index Route Specification](PRD-0100-sigil-web-root-index-route-specification.md):
  Root-route redirect contract for handing operators into `/agents`

### Agents Route

- [PRD-0200 Sigil-Web Agents Route Specification](PRD-0200-sigil-web-agents-route-specification.md):
  Minimal `/agents` route contract centered on selected-agent deep-linking

### Run Detail Route

- [PRD-0300 Sigil-Web Run Detail Route Specification](PRD-0300-sigil-web-run-detail-route-specification.md):
  Owning route-family record for `/runs/$runId` while run-detail behavior is
  re-specified

## Legacy Migration Map

- `PRD-0100-sigil-web-session-and-connection-state-specification.md` -> `PRD-0100-sigil-web-root-index-route-specification.md`
- `PRD-0200-sigil-web-agent-fleet-hub-and-selection-specification.md` -> `PRD-0200-sigil-web-agents-route-specification.md`
- `PRD-0300-sigil-web-run-detail-workspace-specification.md` -> `PRD-0300-sigil-web-run-detail-route-specification.md`
- `PRD-0400-sigil-web-live-orchestration-and-connection-recovery-specification.md` -> retired; future `/runs/$runId` behavior lands under `PRD-0300`
- `PRD-0500-sigil-web-agent-run-control-and-authoring-specification.md` -> retired; future `/agents` or `/runs/$runId` behavior lands under the route-owned PRDs

## Next PRD

Create the next record within the correct route-family block using the first
open number in that block.

Examples:

- `docs/sigil-web/PRD/PRD-0150-<slug>-specification.md` for new root or index route behavior
- `docs/sigil-web/PRD/PRD-0250-<slug>-specification.md` for new `/agents` route behavior
- `docs/sigil-web/PRD/PRD-0350-<slug>-specification.md` for new `/runs/$runId` route behavior
