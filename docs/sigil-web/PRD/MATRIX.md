# MATRIX

## Purpose

This matrix maps `sigil-web` PRD acceptance scenarios to submodule acceptance
scenarios.

## Mapping

| PRD | Scenario ID | PRD Scenario Title | Submodule Feature File | Submodule Scenario Title | Notes |
| --- | --- | --- | --- | --- | --- |
| `PRD-0100` | `SCN-0000` | `Wraps the root agent workspace route in the shared sigil-web application shell` | `sigil-web/acceptance/features/shell.feature` | `Wraps the root agent workspace route in the shared sigil-web application shell` | Shared shell contract for the root workspace |
| `PRD-0100` | `SCN-0001` | `Preserves the shared sigil-web application shell on the standalone run-detail route` | `sigil-web/acceptance/features/shell.feature` | `Preserves the shared sigil-web application shell on the standalone run-detail route` | Shared shell contract for the standalone placeholder route |
| `PRD-0100` | `SCN-0002` | `Exposes the standalone run-detail route as a placeholder workspace frame` | `sigil-web/acceptance/features/shell.feature` | `Exposes the standalone run-detail route as a placeholder workspace frame` | Standalone route remains explicitly placeholder-only |
| `PRD-0200` | `SCN-0000` | `Defaults the root workspace selection to the first available agent and reflects it in the agent search param` | `sigil-web/acceptance/features/workspace.feature` | `Defaults the root workspace selection to the first available agent and reflects it in the agent search param` | Root workspace selects the first available agent |
| `PRD-0200` | `SCN-0001` | `Resolves canonical agent search values with or without the agent_ prefix` | `sigil-web/acceptance/features/workspace.feature` | `Resolves canonical agent search values with or without the agent_ prefix` | Search-param selection accepts raw and canonical values |
| `PRD-0200` | `SCN-0002` | `Updates root workspace selection when a different agent card is chosen` | `sigil-web/acceptance/features/workspace.feature` | `Updates root workspace selection when a different agent card is chosen` | User-driven selection updates the root workspace |
| `PRD-0200` | `SCN-0003` | `Filters agent cards by instance name or endpoint without mutating the underlying fleet` | `sigil-web/acceptance/features/workspace.feature` | `Filters agent cards by instance name or endpoint without mutating the underlying fleet` | Filter affects visibility, not fleet identity |
| `PRD-0210` | `SCN-0000` | `Loads the selected run detail in the root workspace when the selected agent has runs` | `sigil-web/acceptance/features/workspace.feature` | `Loads the selected run detail in the root workspace when the selected agent has runs` | Embedded run detail remains the root default |
| `PRD-0210` | `SCN-0001` | `Shows an empty run-detail prompt when the selected agent has no runs` | `sigil-web/acceptance/features/workspace.feature` | `Shows an empty run-detail prompt when the selected agent has no runs` | Empty detail prompt covers no-run selection |
| `PRD-0210` | `SCN-0002` | `Opens the selected run in the standalone run-detail route from the root workspace` | `sigil-web/acceptance/features/workspace.feature` | `Opens the selected run in the standalone run-detail route from the root workspace` | Root workspace links to the standalone placeholder route |
| `PRD-0300` | `SCN-0000` | `Connects a live agent endpoint and exposes a ready agent session in the fleet` | `sigil-web/acceptance/features/live_sessions.feature` | `Connects a live agent endpoint and exposes a ready agent session in the fleet` | Live session transitions to ready after initialize |
| `PRD-0300` | `SCN-0001` | `Marks a live agent session degraded after missed heartbeats and recovers on reconnect` | `sigil-web/acceptance/features/live_sessions.feature` | `Marks a live agent session degraded after missed heartbeats and recovers on reconnect` | Heartbeat-driven connection state remains visible in the fleet |
| `PRD-0300` | `SCN-0002` | `Applies live runs snapshot and run status notifications to the selected agent workspace` | `sigil-web/acceptance/features/live_sessions.feature` | `Applies live runs snapshot and run status notifications to the selected agent workspace` | Live run notifications update workspace state |
| `PRD-0300` | `SCN-0003` | `Removes a live agent session from the fleet when the operator requests removal` | `sigil-web/acceptance/features/live_sessions.feature` | `Removes a live agent session from the fleet when the operator requests removal` | Operator removal clears the visible live session |
