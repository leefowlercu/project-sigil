# ADR-0001: Sigil-Web Routed Scenario and Paper Design Governance

## Status

Accepted

## Date

2026-03-17

## Context

`sigil-web` needs a UI workflow that is as explicit and reviewable as the
behavior workflow already used for `sigil`. The stack intentionally mixes:

- TanStack Start for route and data orchestration
- ShadCN, Lucide, and Tailwind for the implementation palette
- Paper.Design for page composition and code-aware visual iteration
- Gherkin acceptance scenarios for operator-visible behavior
- Vitest route-contract checks for non-browser verification seams

The first manifest model treated every routed scenario as either visual or
nonvisual inside one Paper-oriented file. That kept parity mechanical, but it
blurred two different concerns:

- routed scenario identity and verification lanes
- Paper-only visual metadata such as artboards, viewports, and selector guards

That binary classification also fit frontend work poorly because a routed
scenario can be both behavioral and visual. Selected-agent deep-linking, for
example, is a route-state contract that benefits from both Vitest and Paper
evidence.

## Decision

Sigil-web adopts these design-governance rules:

- Every routed acceptance scenario MUST map through
  `sigil-web/verification/scenarios/manifest.toml`.
- The scenario manifest MUST carry:
  - `prd`
  - `scenario_id`
  - `title`
  - `route_id`
  - `state_id`
  - fixture names
  - one or more `evidence` entries
- Each `evidence` entry MUST declare a verification lane.
- The currently supported verification lanes are `vitest` and `paper`.
- A routed scenario MAY declare both `vitest` and `paper` evidence when the
  same scenario has meaningful behavioral and visual proof.
- Vitest evidence MUST record a repo-relative test file and stable match string
  in the scenario manifest.
- Paper-specific metadata MUST live in `sigil-web/verification/design/manifest.toml`.
- The design manifest MUST store only Paper-backed scenarios and MUST define:
  - `design_tool`
  - `artboard_name_pattern`
  - required viewports
  - required `data-testid` values
  - per-viewport Paper artboard names
- Paper artboards MUST use the naming convention
  `<prd>--<scenario-id>--<route-id>--<state-id>--<viewport-id>`.
- `@visual` and `@nonvisual` MUST NOT be used in
  `sigil-web/acceptance/features/web_ui.feature`; verification-lane ownership
  now lives in the manifests instead of feature tags.
- ShadCN, Lucide, and Tailwind remain the implementation palette, but they MUST
  reproduce the approved Paper state rather than invent it.

## Decision Details

- Routed UI state identity is defined jointly by PRD scenario title plus
  scenario-manifest route/state mapping, not by ad hoc screenshots in pull
  requests.
- The scenario manifest is the complete routed scenario registry, even for
  scenarios that currently have only Vitest evidence.
- The design manifest is intentionally a Paper-only subset. If a routed
  scenario drops Paper evidence, its design-manifest entry should disappear
  without removing the routed scenario from the scenario manifest.
- `data-testid` values listed in the design manifest are part of the visual
  contract and SHOULD remain stable through visual refactors.
- The initial bootstrap remains desktop-first: only desktop artboards are
  required for Paper-backed scenarios until the design manifest explicitly adds
  more required viewports.
- Paper-authored layouts SHOULD prefer flex layouts and container-driven
  composition so generated JSX can be normalized into TanStack Start and
  ShadCN-friendly code without large semantic rewrites.
- Both manifests are stored as TOML so `./scripts/verify-specs` can parse them
  with standard-library tooling.

## Alternatives Considered

- Keep every routed scenario in one Paper-oriented verification manifest:
  rejected because it makes nonvisual or mixed scenarios feel like second-class
  exceptions inside a design file.
- Treat screenshots in review as the only approval artifact: rejected because
  screenshots are not mechanically traceable to PRDs or scenarios.
- Treat ShadCN blocks as the design source of truth: rejected because block
  libraries accelerate implementation but do not encode product-specific layout
  intent.
- Keep verification metadata outside the repo: rejected because verification
  needs checked-in, reviewable source of truth.

## Consequences

### Positive

- UI state review becomes traceable from PRD to scenario manifest to Paper or
  Vitest evidence to acceptance titles.
- Frontend scenarios can carry both behavioral and visual proof without being
  forced into a single visual or nonvisual bucket.
- Design drift can be caught locally with the same verifier used for PRD/MATRIX
  consistency.

### Negative

- Paper-backed scenarios now require coordination across two manifests instead
  of one combined file.
- Rapid exploratory UI work must still be normalized back into the manifests
  before it is considered complete.

## Related Documents

- [Sigil-Web ADR Index](README.md)
- [PRD-0100 Root and Index Route](../PRD/PRD-0100-sigil-web-root-index-route-specification.md)
- [PRD-0300 Run Detail Route](../PRD/PRD-0300-sigil-web-run-detail-route-specification.md)
- [`sigil-web/verification/scenarios/manifest.toml`](../../../sigil-web/verification/scenarios/manifest.toml)
- [`sigil-web/verification/design/manifest.toml`](../../../sigil-web/verification/design/manifest.toml)
