# GIT

## Commit and Push Authorization

- Do not run `git commit` unless the user explicitly asks for a commit in the current conversation.
- Do not run `git push` unless the user explicitly asks for a push in the current conversation.
- If the user asks for a commit but does not ask for a push, commit only and stop without pushing.
- If the user asks for a push, push only the commits and refs explicitly requested.

## Submodule Pointer Hygiene

- Do not update `sigil` or `sigil-web` submodule pointers unless the task
  requires it.
- When the pointer changes, include a clear reason and expected impact in the change summary.

## Commit Hygiene

- Keep generated files and transient artifacts out of commits.
- Group spec and implementation updates logically so reviewers can trace behavior changes.
