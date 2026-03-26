# ADR

This directory stores architecture decision records for the `sigil-web`
subproject.

- ADRs capture long-lived technical tradeoffs and ownership boundaries.
- Keep ADRs aligned with the active `sigil-web` implementation and PRD suite.
- When an architectural direction changes, update the ADR and the related PRD
  references in the same change.

## Current ADRs

- [`ADR-0001-sigil-web-route-shell-and-workspace-boundaries.md`](ADR-0001-sigil-web-route-shell-and-workspace-boundaries.md):
  Shared shell and route-boundary ownership for the current web surface.
- [`ADR-0002-sigil-web-app-server-session-and-data-boundary.md`](ADR-0002-sigil-web-app-server-session-and-data-boundary.md):
  Session-client, protocol-adapter, and demo/live data boundary.
- [`ADR-0003-sigil-web-agent-browser-acceptance-harness.md`](ADR-0003-sigil-web-agent-browser-acceptance-harness.md):
  Browser-first Gherkin acceptance strategy using `agent-browser`.
