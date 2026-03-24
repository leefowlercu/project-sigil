# ADR-0003: Sigil-Web Generated App-Server Client and Acceptance Lanes

## Status

Accepted

## Date

2026-03-15

## Context

`sigil-web` should not hand-maintain a second copy of the Sigil app-server
protocol. The `sigil` submodule already owns typed protocol definitions and
deterministic generation commands for TypeScript and JSON Schema. The web UI
also needs browser acceptance coverage that stays deterministic during UI work
and exercises live WebSocket behavior through the same scripted fixture shapes
used by local verification.

## Decision

Sigil-web adopts these contract and testing rules:

- TypeScript app-server request, response, and notification types MUST be
  generated from `sigil app-server generate-ts`.
- Browser acceptance MUST remain Gherkin-titled and route through
  `agent-browser`.
- Browser acceptance MUST use a single scripted WebSocket fixture lane for
  deterministic UI-state and error-state coverage.

## Decision Details

- Generated client artifacts SHOULD live under a dedicated `src/lib/appserver`
  boundary once implementation begins.
- `agent-browser` evidence SHOULD be recorded explicitly in
  `sigil-web/verification/scenarios/manifest.toml` with a single
  `lane = "agent-browser"` entry per scenario.
- The scripted fixture lane SHOULD exercise edge states that are awkward or
  slow to provoke through a fully live stack, including protocol
  incompatibility, heartbeat loss, reconnect timing, and malformed input
  errors.
- The scripted fixture lane SHOULD cover the current read-model views used by
  the web UI, including `run/list`, `run/read`, `run/tree/read`,
  `run/steps/list`, `run/events/read`, and `run/subscribe`.
- Acceptance scenario titles remain the behavioral source of truth when the
  scripted fixture lane is used.
- Any future browser lane that targets a live `sigil app-server serve`
  instance is out of scope for this ADR and requires a new ADR or follow-up
  implementation plan.

## Alternatives Considered

- Handwrite WebSocket models in `sigil-web`: rejected because protocol drift
  would be likely and hard to detect.
- Keep a second real-server browser lane: rejected because the extra fixture
  mode and governance complexity are not justified at the current stage of the
  project.
- Keep Playwright as the browser runner: rejected because the project already
  uses `agent-browser` successfully for local live-behavior verification and the
  acceptance harness should align with the tool operators actually use.

## Consequences

### Positive

- Wire contracts stay coupled to the typed server source of truth.
- UI work can move quickly with one deterministic browser acceptance path.
- The scripted fixture becomes the canonical reusable source of live timeline
  browser evidence.

### Negative

- The web UI now depends on an explicit generation step from the `sigil`
  submodule.
- Browser acceptance no longer provides live-stack coverage by default.

## Related Documents

- [Sigil-Web ADR Index](README.md)
- [ADR-0001 Paper Design Governance](ADR-0001-sigil-web-paper-design-governance.md)
- [ADR-0002 TanStack Start Route and State Architecture](ADR-0002-sigil-web-tanstack-start-route-and-state-architecture.md)
- [PRD-0100 Root and Index Route](../PRD/PRD-0100-sigil-web-root-index-route-specification.md)
- [PRD-0200 Agents Route](../PRD/PRD-0200-sigil-web-agents-route-specification.md)
- [PRD-0300 Run Detail Route](../PRD/PRD-0300-sigil-web-run-detail-route-specification.md)
