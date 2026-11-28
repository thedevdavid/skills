# MCP Evals

A portable Agent Skills package for designing, running, and triaging self-owned verification and real-model connector evaluation suites for MCP servers and MCP Apps (Claude, OpenAI, and Google Gemini, with deterministic code/state graders and hard spend guards).

Lives in [`thedevdavid/skills`](https://github.com/thedevdavid/skills) at `skills/mcp-evals/`.

## Install

```sh
npx skills add thedevdavid/skills --skill mcp-evals
```

Or manually:

```sh
mkdir -p ~/.agents/skills
git clone --depth 1 https://github.com/thedevdavid/skills.git /tmp/thedevdavid-skills
cp -R /tmp/thedevdavid-skills/skills/mcp-evals ~/.agents/skills/mcp-evals
```

For Claude Code:

```sh
mkdir -p ~/.claude/skills
ln -s ~/.agents/skills/mcp-evals ~/.claude/skills/mcp-evals
```

## Use

Invoke `$mcp-evals` in Codex or `/mcp-evals` in compatible hosts, and specify the target MCP server, verification tier, or eval run to build or triage.

Workflow: [SKILL.md](SKILL.md). References:
- [references/protocol-and-authority.md](references/protocol-and-authority.md)
- [references/vendor-drivers-and-spend.md](references/vendor-drivers-and-spend.md)
- [references/grading-and-scenarios.md](references/grading-and-scenarios.md)
- [references/triage-compare-and-import.md](references/triage-compare-and-import.md)
