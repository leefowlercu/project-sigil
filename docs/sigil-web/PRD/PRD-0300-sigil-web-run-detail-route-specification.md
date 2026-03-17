# PRD-0300: Sigil-Web Run Detail Route Specification

## Status

Draft

## Context

The `/runs/$runId` route family remains the route-family owner for run-detail
behavior in `sigil-web`. The route boundary still matters to the application
architecture even though the operator-visible behavior for this route is being
re-specified from a clean slate.

## Goals

- Preserve ownership of the `/runs/$runId` route family.
- Keep a stable place for future run-detail acceptance scenarios.

## Non-Goals

- Defining current run-detail workspace behavior.
- Defining live orchestration, run-start, run-stop, or drill-in behavior.

## Route Contract

- The `/runs/$runId` route family MUST inherit the application-shell layout
  contract defined in `PRD-0150`.
- The `/runs/$runId` route family remains the owning route family for future
  run-detail behavior.
- No acceptance scenarios are currently defined for this route family; behavior
  is intentionally being re-specified from a clean slate.
