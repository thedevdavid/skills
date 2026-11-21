# Framework upgrade patterns

Read when Next.js, React, TypeScript, or ESLint are in the bump set. These are **sequencing rules**, not compatibility matrices — always prefer the live official upgrade guide for the crossed versions.

## Next.js

- Research **every adjacent major** guide on the path (e.g. 14→15 and 15→16), even if a CLI can jump straight to the endpoint.
- Run the official Next codemod/migrator for each step it supports; treat `@next-codemod-error` / leftover TODOs as open ledger rows, not done.
- Async request-API / config shims often collapse across majors — do not assume a 15-era shim still works on 16 without reading both guides.

## React

- Prefer the documented **bridge** minor when upgrading toward a new major (e.g. land on 18.3 and clear its warnings before 19) when the upgrade guide says so.
- Pair `react` / `react-dom` (and `@types/react` / `@types/react-dom` when used) in the same wave.
- Codemods cover a subset; remaining classic-runtime / deprecated-API call sites stay `open` until grepped clean.

## TypeScript

- Treat each `x.y` release as potentially break-equivalent for compile: read Breaking Changes notes for **every** intermediate `x.y` on the path, not only the endpoint.
- Minimum `typescript` peer bumps required by frameworks belong on the ledger as `required-config` or `breaking` as sources say.

## ESLint

1. Run the official **migrator / codemod** for the major (e.g. flat config migration) first.
2. Then `eslint --fix` (or project equivalent) for auto-fixable residuals.
3. Leftover rule / config breakages are required ledger rows — do not silence with broad disables to “finish” the upgrade.

## Shared

Prefer official migrators before hand edits. Endpoint install success ≠ migration completeness across intermediate majors.
