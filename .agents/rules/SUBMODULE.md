# SUBMODULE

## Working in `sigil` and `sigil-web`

- `sigil/` is a git submodule.
- `sigil-web/` is a git submodule.
- Treat `sigil/AGENTS.md` as the authoritative instruction set for work scoped
  to submodule files when that file exists.
- Treat `sigil-web/AGENTS.md` as the authoritative instruction set for work
  scoped to `sigil-web/` files when that file exists.
- If a submodule has no `AGENTS.md`, use that submodule's `README.md` and any
  local `.agents/rules/` files as the local guidance source.
- Apply local rules in each submodule's `.agents/rules/` directory for
  implementation work when those rules exist.

## Coordination

- Keep specs (`docs/<subproject>/`) and implementation (`sigil/` or
  `sigil-web/`) aligned in the same change sequence.
- If a spec change is intentionally not yet implemented, state that gap explicitly in handoff notes.
