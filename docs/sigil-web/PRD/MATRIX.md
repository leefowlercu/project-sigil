# MATRIX

## Purpose

This matrix maps `sigil-web` PRD acceptance scenarios to submodule acceptance
scenarios.

## Mapping

| PRD | Scenario ID | PRD Scenario Title | Submodule Feature File | Submodule Scenario Title | Notes |
| --- | --- | --- | --- | --- | --- |
| `PRD-0100` | `SCN-0000` | `Redirects the root route to the agents hub` | `sigil-web/acceptance/features/web_ui.feature` | `Redirects the root route to the agents hub` | Structural redirect from `/` into the primary `/agents` route family |
| `PRD-0200` | `SCN-0000` | `Deep-links the selected agent in the agents route` | `sigil-web/acceptance/features/web_ui.feature` | `Deep-links the selected agent in the agents route` | Minimal preserved `/agents?agent=<id>` route-state contract |

`PRD-0300` currently has no mapped acceptance scenarios; the `/runs/$runId`
route family remains intentionally unspecced while a new behavior set is
authored.
