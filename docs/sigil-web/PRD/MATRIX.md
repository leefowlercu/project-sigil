# MATRIX

## Purpose

This matrix maps `sigil-web` PRD acceptance scenarios to submodule acceptance
scenarios.

## Mapping

| PRD | Scenario ID | PRD Scenario Title | Submodule Feature File | Submodule Scenario Title | Notes |
| --- | --- | --- | --- | --- | --- |
| `PRD-0100` | `SCN-0000` | `Renders the primary agent workspace at the root route` | `sigil-web/acceptance/features/web_ui.feature` | `Renders the primary agent workspace at the root route` | Root route owns the primary agent workspace without a redirect handoff |
| `PRD-0100` | `SCN-0001` | `Deep-links the selected agent in the root route` | `sigil-web/acceptance/features/web_ui.feature` | `Deep-links the selected agent in the root route` | Root route preserves the selected-agent search-state contract as `/?agent=<id>` |
| `PRD-0150` | `SCN-0000` | `Keeps routed workspaces inside the application shell on supported desktop viewports` | `sigil-web/acceptance/features/web_ui.feature` | `Keeps routed workspaces inside the application shell on supported desktop viewports` | Root application shell owns desktop-height workspace containment and pane-local overflow |
| `PRD-0150` | `SCN-0001` | `Preserves access below the minimum supported desktop height` | `sigil-web/acceptance/features/web_ui.feature` | `Preserves access below the minimum supported desktop height` | Root application shell owns the compact-height fallback when the desktop viewport minimum is not met |
| `PRD-0300` | `SCN-0000` | `Auto-follows the latest event while a live run detail timeline is pinned to the bottom` | `sigil-web/acceptance/features/web_ui.feature` | `Auto-follows the latest event while a live run detail timeline is pinned to the bottom` | Shared run-detail timeline keeps the latest live event visible while the operator remains pinned to the bottom |
| `PRD-0300` | `SCN-0001` | `Shows a centered scroll-to-bottom control and resumes follow when the operator has scrolled away from the latest live event` | `sigil-web/acceptance/features/web_ui.feature` | `Shows a centered scroll-to-bottom control and resumes follow when the operator has scrolled away from the latest live event` | Shared run-detail timeline reveals a centered recovery control and re-arms live follow after the operator returns to the latest event |
