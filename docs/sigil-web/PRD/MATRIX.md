# MATRIX

## Purpose

This matrix maps `sigil-web` PRD acceptance scenarios to submodule acceptance scenarios.

## Mapping

| PRD | Scenario ID | PRD Scenario Title | Submodule Feature File | Submodule Scenario Title | Notes |
| --- | --- | --- | --- | --- | --- |
| `PRD-0100` | `SCN-0000` | `Connects to a compatible app-server and enters a ready agents hub state` | `sigil-web/acceptance/features/web_ui.feature` | `Connects to a compatible app-server and enters a ready agents hub state` | Session and handshake now land in the fleet hub |
| `PRD-0100` | `SCN-0001` | `Blocks the agents hub with an incompatible-server state when protocol negotiation fails` | `sigil-web/acceptance/features/web_ui.feature` | `Blocks the agents hub with an incompatible-server state when protocol negotiation fails` | Compatibility gate blocks the hub rather than a standalone connect page |
| `PRD-0100` | `SCN-0002` | `Marks the session degraded and begins reconnecting without discarding current agent selection intent` | `sigil-web/acceptance/features/web_ui.feature` | `Marks the session degraded and begins reconnecting without discarding current agent selection intent` | Reconnect preserves hub selection intent |
| `PRD-0200` | `SCN-0000` | `Lists connected agents in the command hub after session initialization` | `sigil-web/acceptance/features/web_ui.feature` | `Lists connected agents in the command hub after session initialization` | Fleet hub baseline |
| `PRD-0200` | `SCN-0001` | `Deep-links the selected agent in the agents hub and shows its details plus runs` | `sigil-web/acceptance/features/web_ui.feature` | `Deep-links the selected agent in the agents hub and shows its details plus runs` | Encodes `/agents?agent=<id>` selection state |
| `PRD-0200` | `SCN-0002` | `Shows a fleet empty state when no agents are connected to the app-server` | `sigil-web/acceptance/features/web_ui.feature` | `Shows a fleet empty state when no agents are connected to the app-server` | Fleet empty-state baseline |
| `PRD-0300` | `SCN-0000` | `Opens a run detail workspace from the agents hub with summary tree and timeline panes` | `sigil-web/acceptance/features/web_ui.feature` | `Opens a run detail workspace from the agents hub with summary tree and timeline panes` | Run-detail shell launched from selected-agent context |
| `PRD-0300` | `SCN-0001` | `Selects a node step and typed artifact without leaving the current run workspace` | `sigil-web/acceptance/features/web_ui.feature` | `Selects a node step and typed artifact without leaving the current run workspace` | Bootstrap drill-in baseline |
| `PRD-0300` | `SCN-0002` | `Refreshes run detail data without losing the current selection context` | `sigil-web/acceptance/features/web_ui.feature` | `Refreshes run detail data without losing the current selection context` | Bootstrap refresh-state baseline |
| `PRD-0400` | `SCN-0000` | `Attaches a live run workspace to canonical subscription updates` | `sigil-web/acceptance/features/web_ui.feature` | `Attaches a live run workspace to canonical subscription updates` | Bootstrap live-subscribe baseline |
| `PRD-0400` | `SCN-0001` | `Resumes a live run workspace after reconnect without duplicating applied events` | `sigil-web/acceptance/features/web_ui.feature` | `Resumes a live run workspace after reconnect without duplicating applied events` | Bootstrap resume-afterSeq baseline |
| `PRD-0400` | `SCN-0002` | `Transitions a subscribed run workspace into terminal completed or interrupted state` | `sigil-web/acceptance/features/web_ui.feature` | `Transitions a subscribed run workspace into terminal completed or interrupted state` | Bootstrap terminal live-state baseline |
| `PRD-0500` | `SCN-0000` | `Starts a run for the selected agent from an agents-hub dialog without requiring server-local file paths` | `sigil-web/acceptance/features/web_ui.feature` | `Starts a run for the selected agent from an agents-hub dialog without requiring server-local file paths` | New-run dialog now belongs to the selected-agent hub |
| `PRD-0500` | `SCN-0001` | `Validates malformed inline YAML in the selected agent's new run dialog before issuing run start` | `sigil-web/acceptance/features/web_ui.feature` | `Validates malformed inline YAML in the selected agent's new run dialog before issuing run start` | Dialog-local composer validation baseline |
| `PRD-0500` | `SCN-0002` | `Stops an active run and surfaces interrupted terminal state plus stop provenance` | `sigil-web/acceptance/features/web_ui.feature` | `Stops an active run and surfaces interrupted terminal state plus stop provenance` | Bootstrap run-stop UX baseline |
