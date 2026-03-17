# PRD-0150: Sigil-Web Application Shell Layout Specification

## Status

Draft

## Context

The root route in `sigil-web` owns the application shell that wraps every
routed operator workspace. Operators should experience the shell as a
desktop-style control surface rather than a long-form document page, while the
product still preserves a safe fallback when the viewport height drops below
the desktop target.

## Goals

- Define the application-wide layout contract owned by the root application
  shell.
- Keep supported desktop workspaces constrained to one viewport with pane-local
  overflow.
- Define the minimum desktop viewport height and the compact-height fallback
  behavior.

## Non-Goals

- Defining `/agents` route content or selection behavior in detail.
- Defining `/runs/$runId` route content or live orchestration behavior in
  detail.
- Defining mobile-first layout behavior beyond the compact-height fallback.

## Application Shell Contract

- `sigil-web` MUST render routed operator workspaces inside a viewport-constrained
  application shell owned by the root route.
- On desktop viewports at or above `1280x800` CSS pixels, the application shell
  MUST fit within a single viewport without document-level vertical scrolling.
- Overflowing routed content MUST scroll within designated workspace panes
  instead of increasing document height.
- The root application shell MUST define a minimum supported desktop viewport
  height of `800` CSS pixels.
- Below `800` CSS pixels of viewport height, the application MAY switch to a
  compact layout or allow document-level vertical scrolling, but it MUST
  preserve access to primary navigation, primary context, and primary actions.

## Acceptance Scenarios

### Scenario SCN-0000: Keeps routed workspaces inside the application shell on supported desktop viewports

Given the operator opens a routed sigil-web workspace on a desktop viewport at
least `1280` CSS pixels wide and `800` CSS pixels tall  
When the application renders the root application shell  
Then the document does not scroll vertically  
And overflowing route content scrolls inside workspace panes.

### Scenario SCN-0001: Preserves access below the minimum supported desktop height

Given the operator opens a routed sigil-web workspace on a viewport shorter
than `800` CSS pixels  
When the application applies its compact-height fallback  
Then the operator can still reach primary navigation, primary context, and
primary actions.
