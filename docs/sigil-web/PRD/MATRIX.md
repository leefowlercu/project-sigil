# MATRIX

## Purpose

This matrix maps `sigil-web` PRD acceptance scenarios to submodule acceptance scenarios.

## Mapping

| PRD | Scenario ID | PRD Scenario Title | Submodule Feature File | Submodule Scenario Title | Notes |
| --- | --- | --- | --- | --- | --- |
| `PRD-0100` | `SCN-0000` | `Connects to a compatible app-server and enters ready operator state` | `sigil-web/acceptance/features/web_ui.feature` | `Connects to a compatible app-server and enters ready operator state` | Bootstrap session and handshake UX baseline |
| `PRD-0100` | `SCN-0001` | `Blocks the workspace with an incompatible-server state when protocol negotiation fails` | `sigil-web/acceptance/features/web_ui.feature` | `Blocks the workspace with an incompatible-server state when protocol negotiation fails` | Bootstrap compatibility gate baseline |
| `PRD-0100` | `SCN-0002` | `Marks the session degraded and begins reconnecting after a missed heartbeat` | `sigil-web/acceptance/features/web_ui.feature` | `Marks the session degraded and begins reconnecting after a missed heartbeat` | Bootstrap reconnect UX baseline |
| `PRD-0200` | `SCN-0000` | `Lists persisted runs after session initialization` | `sigil-web/acceptance/features/web_ui.feature` | `Lists persisted runs after session initialization` | Bootstrap run-list baseline |
| `PRD-0200` | `SCN-0001` | `Paginates the run list without losing current connection context` | `sigil-web/acceptance/features/web_ui.feature` | `Paginates the run list without losing current connection context` | Bootstrap run-list pagination baseline |
| `PRD-0200` | `SCN-0002` | `Shows an empty-state call to action when the run corpus has no runs` | `sigil-web/acceptance/features/web_ui.feature` | `Shows an empty-state call to action when the run corpus has no runs` | Bootstrap run-list empty-state baseline |
| `PRD-0300` | `SCN-0000` | `Opens a run detail workspace with summary tree and timeline panes` | `sigil-web/acceptance/features/web_ui.feature` | `Opens a run detail workspace with summary tree and timeline panes` | Bootstrap run-detail shell baseline |
| `PRD-0300` | `SCN-0001` | `Selects a node step and typed artifact without leaving the current run workspace` | `sigil-web/acceptance/features/web_ui.feature` | `Selects a node step and typed artifact without leaving the current run workspace` | Bootstrap drill-in baseline |
| `PRD-0300` | `SCN-0002` | `Refreshes run detail data without losing the current selection context` | `sigil-web/acceptance/features/web_ui.feature` | `Refreshes run detail data without losing the current selection context` | Bootstrap refresh-state baseline |
| `PRD-0400` | `SCN-0000` | `Attaches a live run workspace to canonical subscription updates` | `sigil-web/acceptance/features/web_ui.feature` | `Attaches a live run workspace to canonical subscription updates` | Bootstrap live-subscribe baseline |
| `PRD-0400` | `SCN-0001` | `Resumes a live run subscription after reconnect without duplicating applied events` | `sigil-web/acceptance/features/web_ui.feature` | `Resumes a live run subscription after reconnect without duplicating applied events` | Bootstrap resume-afterSeq baseline |
| `PRD-0400` | `SCN-0002` | `Transitions a subscribed run workspace into terminal completed or interrupted state` | `sigil-web/acceptance/features/web_ui.feature` | `Transitions a subscribed run workspace into terminal completed or interrupted state` | Bootstrap terminal live-state baseline |
| `PRD-0500` | `SCN-0000` | `Starts a run from an inline YAML composer without requiring server-local file paths` | `sigil-web/acceptance/features/web_ui.feature` | `Starts a run from an inline YAML composer without requiring server-local file paths` | Bootstrap run-start composer baseline |
| `PRD-0500` | `SCN-0001` | `Validates malformed inline YAML before issuing run start` | `sigil-web/acceptance/features/web_ui.feature` | `Validates malformed inline YAML before issuing run start` | Bootstrap composer validation baseline |
| `PRD-0500` | `SCN-0002` | `Stops an active run and surfaces interrupted terminal state plus stop provenance` | `sigil-web/acceptance/features/web_ui.feature` | `Stops an active run and surfaces interrupted terminal state plus stop provenance` | Bootstrap run-stop UX baseline |
