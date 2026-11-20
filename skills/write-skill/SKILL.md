---
name: write-skill
description: "Guides creation of a new Agent Skill from a real workflow or session. Use when extracting a skill, writing SKILL.md, naming or describing a skill, structuring references/, or turning corrections into durable agent instructions."
license: MIT
metadata:
  author: thedevdavid
  version: "1.0"
---

# Write Skill

Extract a reusable Agent Skill from real expertise — a completed session, a correction loop, or judgment already encoded in an existing high-bar skill. Deliver a portable package (`SKILL.md` + optional `references/`, `scripts/`, `agents/`, README) that any Agent Skills–compatible host can install. Generic “best practices” prose without source material is not a skill.

Density target: match the `prune-tests` skill in this collection (monorepo path `skills/prune-tests/`) — explicit gates, evidence language, anti-filler. Motivational essays and host lock-in fail this skill.

## Discovery (do not skip)

Before naming or outlining, gather **source material**. Require at least one of:

1. A real session/correction where the agent was steered (what worked, what was wrong, preferred tools/constraints).
2. Existing project artifacts that encode judgment (runbooks, review comments, incident notes, a skill like prune-tests).
3. An explicit user statement of the judgment to encode.

If none exists, stop and ask for source material. Do not invent a skill from model priors alone.

Clarify and record (temporary notes outside the repo unless the user requires a committed artifact):

| Field | Lock |
| --- | --- |
| Purpose | One coherent unit of work the skill owns |
| When-to-use | Concrete trigger situations and phrases |
| Non-goals | Adjacent work this skill must not absorb |
| Source | Session, artifact, or stated judgment |
| Invocation | Auto via description vs slash-only preference |
| Overlap | Existing skills in this monorepo, product packs (e.g. second-skills), or better upstream installs |

Ask the user only when a choice **blocks Design** (ambiguous purpose, conflicting non-goals, or unclear auto vs slash). Do not interview for taste.

Read [david-conventions.md](references/david-conventions.md) when judging whether the source is dense enough to extract, or whether the topic belongs elsewhere.

## Design

Lock these before writing files:

1. **`name`** — kebab-case; equals folder name; ≤64 chars; `[a-z0-9-]` only; no leading/trailing/consecutive hyphens.
2. **`description`** — third person; **what + when**; concrete trigger terms; ≤1024 chars. This is the routing key for hosts that select by metadata. Vague slogans fail. Read [description-guide.md](references/description-guide.md) when drafting or stress-testing the description.
3. **Body outline** — section list for `SKILL.md`; which judgment gates stay inline vs move to `references/` with “read when …” cues.
4. **Invocation** — prefer description-triggered auto selection when the when-to-use is crisp (create/extract/write a skill is such a case). Prefer slash-only when false positives are likely. Optional host-specific frontmatter (e.g. Cursor `disable-model-invocation`) may document that choice; it must not be required for the skill to work on other hosts.
5. **Layout** — monorepo path `skills/<name>/`. Never author into built-in host skill caches (e.g. Cursor’s `~/.cursor/skills-cursor/`). Consumers install via `npx skills` or copy into shared/`~/.agents/skills` (and host-native skill dirs).

Reject Design if purpose is a mega process framework, a commodity skill better installed upstream, or product-pack content that belongs in a separate repo.

## Implementation

Create:

```
skills/<name>/
├── SKILL.md                 # required
├── README.md                # multi-host install; optional but preferred
├── agents/openai.yaml       # optional Codex UI glue
├── references/…             # optional; one level deep from SKILL.md
└── scripts/…                # only when the same logic was reinvented across runs
```

### SKILL.md body rules

- Teach **how to approach** the class of problems, not the answer to one instance.
- Put **gotchas** inline — concrete corrections to mistakes the agent will make without being told.
- Keep the body **lean** (under ~500 lines). Move deep examples, matrices, and long checklists to `references/` with one-level links and load cues (“Read X when …”).
- **Scripts** only when execution traces show the agent reinventing the same logic; document deps; keep portable (shell/Python). Otherwise prose + references.
- **No tests or fixtures** inside `skills/<name>/` — installers copy the whole folder.
- Soft metadata when shipping: `license: MIT`, `metadata.author: thedevdavid`, `metadata.version`.
- Body language must be **host-agnostic**. No “only in Cursor”, no required Cursor-only tools/MCP/APIs. Host install tips belong in README / optional `agents/` metadata.

Read [structure-checklist.md](references/structure-checklist.md) before finalizing file layout or splitting references.

### Encode judgment, not vibes

Require the same class of gates prune-tests uses for retention:

- Case-by-case decisions with evidence, not slogans.
- Temporary decision records outside the repo by default.
- Anti-filler: delete instructional prose the agent already knows; keep what it would get wrong without the skill.
- Retention/evidence vocabulary over confidence theater.

Point authors at `skills/prune-tests/` as the density bar for this monorepo.

### Catalog and install surface

Update the root README catalog row. Skill README should cover `npx skills add <owner>/skills --skill <name>` plus shared `~/.agents/skills` (noting Claude/Codex/Grok as consumers of that layout). Do not make a single-host marketplace wrapper the core delivery.

## Verification

Before calling the skill done, run this gate (hand off to `review-skill` when it exists):

1. **Source** — extracted from real judgment/session, not LLM-generic filler.
2. **Description smoke** — would a user utterance about the workflow select this skill by name/description? Would unrelated chat avoid it?
3. **Frontmatter** — valid `name`/`description`; soft metadata present; any host-only fields optional and documented.
4. **Size / disclosure** — `SKILL.md` under ~500 lines; references one-level-deep with load cues.
5. **Installer safety** — no tests/fixtures under `skills/<name>/`.
6. **Portability** — plain Agent Skills package; body makes sense on Cursor, Codex, Claude, Grok, and peers without required proprietary APIs.
7. **Overlap** — no accidental second-skills or upstream commodity duplicate.
8. **Density** — comparable to prune-tests; explicit gates; anti-filler.
9. **Catalog** — root README lists the skill.
10. **Dry run** — prefer one real-task pass; feed corrections into gotchas.

Complete when the folder passes the gate, the catalog is updated, and remaining gaps are disclosed. Do not keep polishing prose to meet a line count or aesthetic. Stop after one verification pass unless the dry run surfaces a concrete gotcha to encode.
