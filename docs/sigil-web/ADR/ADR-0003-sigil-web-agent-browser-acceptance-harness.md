# ADR-0003: Sigil-Web Agent-Browser Acceptance Harness

## Status

Accepted

## Context

The `sigil-web` bootstrap needs a browser-first acceptance lane that matches the
spec-driven workflow already used in `sigil`, while using the workstation's
existing `agent-browser` tooling instead of a second browser-automation stack.

## Decision

- Gherkin feature files remain the behavioral source of truth for
  `sigil-web` acceptance.
- Acceptance execution uses `@cucumber/cucumber` plus the globally installed
  `agent-browser` CLI.
- `agent-browser` is treated as an external prerequisite on `PATH`, with
  `AGENT_BROWSER_BIN` as an override for non-default installations.
- Acceptance helpers must isolate browser sessions, capture failure artifacts,
  and fail fast when `agent-browser` is unavailable.

## Consequences

- The repository documents the `agent-browser` prerequisite instead of
  vendoring another browser driver.
- Browser interaction helpers become a shared test harness surface and should
  remain thin wrappers over `agent-browser` commands.

## Related Documents

- [`../PRD/PRD-0300-sigil-web-live-app-server-session-and-run-subscription-specification.md`](../PRD/PRD-0300-sigil-web-live-app-server-session-and-run-subscription-specification.md)
