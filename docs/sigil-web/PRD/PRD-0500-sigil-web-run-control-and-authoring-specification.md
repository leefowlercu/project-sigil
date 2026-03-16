# PRD-0500: Sigil-Web Run Control and Authoring Specification

## Status

Draft

## Context

`sigil-web` should let operators start runs from inline YAML and stop active
runs without dropping down to the CLI, while still preserving the same backend
control semantics exposed by the Sigil app-server.

## Goals

- Define the inline YAML run-start workflow.
- Define client-side validation expectations for malformed YAML.
- Define operator-visible stop control outcomes.

## Non-Goals

- Defining template library management or saved drafts.
- Defining alternate transport or deployment policy.

## Run Start Contract

- The UI MUST expose an inline YAML composer for `run/start`.
- Run-start UI MUST not require server-local file paths.
- The composer SHOULD support operator-visible template variable entry when the
  underlying start surface needs it.
- The UI SHOULD validate clearly malformed inline YAML before issuing
  `run/start`.
- Successful run-start UI MUST transition the operator into the newly created
  run context.

## Run Stop Contract

- The UI MUST expose stop control only when the selected run is in scope for the
  configured app-server run corpus.
- Stop control MUST surface the resulting terminal status and stop provenance
  returned by the app-server contract.
- Terminal runs MUST surface the resulting no-op or already-terminal outcome
  without implying a second interruption occurred.

## Acceptance Scenarios

### Scenario SCN-0000: Starts a run from an inline YAML composer without requiring server-local file paths

Given the session is ready and the operator opens the run authoring workflow  
When the operator submits valid inline YAML through the composer  
Then the application starts the run without requiring server-local file paths
and moves into the new run context.

### Scenario SCN-0001: Validates malformed inline YAML before issuing run start

Given the session is ready and the operator opens the run authoring workflow  
When the operator enters malformed inline YAML  
Then the application shows validation feedback before issuing `run/start`.

### Scenario SCN-0002: Stops an active run and surfaces interrupted terminal state plus stop provenance

Given the session is ready and the operator is viewing an active run  
When the operator requests run stop and the app-server accepts the stop  
Then the application surfaces interrupted terminal state plus stop provenance
for that run.
