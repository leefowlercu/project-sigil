# ADR-0001: Sigil-Web Route Shell and Workspace Boundaries

## Status

Accepted

## Context

`sigil-web` currently exposes a narrow route surface with a shared application
shell, a root agent workspace, and a standalone run-detail placeholder route.

The spec suite needs a durable ownership boundary for which layers own:

- shared shell chrome
- route-level workspace containers
- route-local behavior

## Decision

- The root route owns the shared application shell and shared document chrome.
- The `/` route owns the primary operator workspace, including fleet selection,
  run list, and embedded run detail.
- The `/runs/$runId` route remains a standalone run-detail placeholder until a
  future PRD replaces that contract.
- Shared shell selectors and route workspace containers are product contracts
  and must remain acceptance-backed when the layout changes.

## Consequences

- Shell and route-container changes must update the owning PRD, matrix rows, and
  acceptance scenarios together.
- Future standalone run-detail work must replace the placeholder contract
  explicitly instead of silently expanding it in implementation.

## Related Documents

- [`../PRD/PRD-0100-sigil-web-application-shell-and-route-surface-specification.md`](../PRD/PRD-0100-sigil-web-application-shell-and-route-surface-specification.md)
- [`../PRD/PRD-0210-sigil-web-run-detail-workspace-specification.md`](../PRD/PRD-0210-sigil-web-run-detail-workspace-specification.md)
