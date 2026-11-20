# Axes detail

Read when scoring Standards smells or Intent coverage.

## Standards — thin smell baseline

Heuristics only (never hard fails when the repo endorses the pattern). Skip linter/formatter territory.

| Smell | Signal in the diff |
| --- | --- |
| Duplicated logic | Same shape copied across hunks/files |
| Feature envy | Method lives on the wrong type’s data |
| Shotgun / divergent change | One concern edits many unrelated modules, or one module edits for many reasons |
| Speculative generality | Abstraction/hooks for needs the ask does not have |
| Mysterious names | Names that hide what the symbol does |

Repo standards win over this list.

## Intent checklist

- [ ] Each stated requirement is present, partial, or explicitly out of scope
- [ ] Behaviour matches the ask (not only “related”)
- [ ] Drive-by refactors/features called out as scope creep
- [ ] “Looks done” without touching the claimed surface → fail Intent

No ask/spec → Intent = unresolved; do not hallucinate requirements.
