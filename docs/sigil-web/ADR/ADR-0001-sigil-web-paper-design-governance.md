# ADR-0001: Sigil-Web Paper Design Governance

## Status

Accepted

## Date

2026-03-15

## Context

`sigil-web` needs a UI workflow that is as explicit and reviewable as the
behavior workflow already used for `sigil`. The stack intentionally mixes:

- TanStack Start for route and data orchestration
- ShadCN, Lucide, and Tailwind for the implementation palette
- Paper.Design for page composition and code-aware visual iteration
- Gherkin acceptance scenarios for operator-visible behavior

Without an explicit governance rule, visual intent will drift away from the
PRDs and acceptance suite even when behavior remains correct.

## Decision

Sigil-web adopts these design-governance rules:

- Paper artboards are the normative visual source for routed UI states.
- Every routed acceptance scenario MUST map to Paper artboards through
  `sigil-web/design/design-manifest.toml`.
- The design manifest MUST carry:
  - `prd`
  - `scenario_id`
  - `title`
  - `route_id`
  - `state_id`
  - required Paper artboard names for the currently enforced viewports
  - fixture names
  - required `data-testid` values
- Artboards MUST use the naming convention
  `<prd>--<scenario-id>--<route-id>--<state-id>--<viewport-id>`.
- ShadCN, Lucide, and Tailwind remain the implementation palette, but they MUST
  reproduce the approved Paper state rather than invent it.

## Decision Details

- Routed UI states are defined jointly by PRD scenario title and design-manifest
  state mapping, not by ad hoc screenshots in pull requests.
- `data-testid` values listed in the design manifest are part of the acceptance
  contract and SHOULD remain stable through visual refactors.
- The initial bootstrap is desktop-first: only desktop artboards are required
  until the manifest explicitly adds more required viewports.
- Paper-authored layouts SHOULD prefer flex layouts and container-driven
  composition so generated JSX can be normalized into TanStack Start and
  ShadCN-friendly code without large semantic rewrites.
- The design manifest is stored as TOML instead of YAML so
  `./scripts/verify-specs` can parse it with standard-library tooling.

## Alternatives Considered

- Treat screenshots in review as the only design approval artifact: rejected
  because screenshots are not mechanically traceable to PRDs or scenarios.
- Treat ShadCN blocks as the design source of truth: rejected because block
  libraries accelerate implementation but do not encode product-specific layout
  intent.
- Keep the design manifest outside the repo: rejected because verification needs
  a checked-in source of truth.

## Consequences

### Positive

- UI state review becomes traceable from PRD to Paper to acceptance titles.
- Design drift can be caught locally with the same verifier used for PRD/MATRIX
  consistency.
- The implementation can still move quickly because Paper remains code-aware.

### Negative

- Every routed scenario now carries additional manifest and artboard
  maintenance cost.
- Rapid exploratory UI work must be normalized back into the manifest before it
  is considered complete.

## Related Documents

- [Sigil-Web ADR Index](README.md)
- [PRD-0100 Session and Connection State](../PRD/PRD-0100-sigil-web-session-and-connection-state-specification.md)
- [PRD-0200 Operator Shell and Run List](../PRD/PRD-0200-sigil-web-operator-shell-and-run-list-specification.md)
- [PRD-0300 Run Detail Workspace](../PRD/PRD-0300-sigil-web-run-detail-workspace-specification.md)
- [PRD-0400 Live Orchestration and Connection Recovery](../PRD/PRD-0400-sigil-web-live-orchestration-and-connection-recovery-specification.md)
- [PRD-0500 Run Control and Authoring](../PRD/PRD-0500-sigil-web-run-control-and-authoring-specification.md)
- [`sigil-web/design/design-manifest.toml`](../../../sigil-web/design/design-manifest.toml)
