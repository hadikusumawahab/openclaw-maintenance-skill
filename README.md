# OpenClaw Maintenance Skill

Best practices in a single `CLAUDE.md` file to maintain your OpenClaw agents with ease. Works for Codex CLI and Gemini CLI too.

## The Problem

Running multiple OpenClaw agents sounds simple until workspace files bloat, memory drifts, config changes silently break things, and agents forget what they learned last week.

Proper maintenance is the difference between agents that degrade over time and agents that stay sharp.

Common issues after a few weeks of running OpenClaw:

- AGENTS.md hits the 20K character limit — rules at the bottom get silently truncated
- Multiple agents have the same instructions copy-pasted, each slightly different
- Agent makes the same mistake you corrected last Tuesday
- After OpenClaw updates, config fields may behave differently — unverified setups risk silent failures

## The Solution

Five principles that directly address these issues:

| Principle | Addresses |
|-----------|-----------|
| **Lean Workspace** | Bloated files, silent truncation, noisy context |
| **Skills Over Bloat** | Duplicated instructions across agents, workspace sprawl |
| **Design for Continuity** | Forgotten decisions, repeated mistakes, lessons that never stick |
| **Trust but Verify** | Silent config drift, stale docs, version mismatches |
| **Design for Agent Coherence** | Cross-agent inconsistencies, scope overlap, diverged shared instructions |

## The Five Principles

### 1. Lean Workspace

Every line in a workspace file costs tokens every turn. OpenClaw enforces hard limits (20K chars per file, 150K total) — files that exceed the limit are silently truncated. Keep files focused, separate agent-specific from shared sections, and prune regularly.

### 2. Skills Over Bloat

When you notice the same instructions duplicated across agents, extract them into a shared skill. Skills use progressive disclosure — only metadata in context, full instructions load on demand. One canonical location, never duplicate.

### 3. Design for Continuity

Agents forget everything between sessions. When they keep forgetting or repeating mistakes, the problem is usually that important context lives in chat instead of files. Diagnose by checking if decisions are persisted, prune bloated memory, and design self-learning triggers so corrections become permanent rules.

### 4. Trust but Verify

OpenClaw ships frequent updates — config fields get renamed, CLI flags change, features behave differently between versions. Trust your setup, but verify after every change. Fetch current docs before assuming. Run diagnostics after config edits.

### 5. Design for Agent Coherence

Before changing one agent, understand the whole system. When maintaining multiple agents, changes to one can create inconsistencies across others. Read scope boundaries, check for duplicate skills, and treat shared instructions as shared content — not copy-pasted sections that diverge over time.

## Install

### Claude Code (Plugin)

From within Claude Code, add the marketplace and install:

```
/plugin marketplace add hadikusumawahab/openclaw-maintenance-skill
/plugin install openclaw-maintenance-skill@openclaw-maintenance-skill
```

### Claude Code (Per-Project)

New project:

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/hadikusumawahab/openclaw-maintenance-skill/main/CLAUDE.md
```

Existing project (append):

```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/hadikusumawahab/openclaw-maintenance-skill/main/CLAUDE.md >> CLAUDE.md
```

### Codex CLI

```bash
curl -o AGENTS.md https://raw.githubusercontent.com/hadikusumawahab/openclaw-maintenance-skill/main/AGENTS.md
```

### Gemini CLI

```bash
curl -o GEMINI.md https://raw.githubusercontent.com/hadikusumawahab/openclaw-maintenance-skill/main/GEMINI.md
```

## How to Know It's Working

- Workspace files stay lean across months, not growing unbounded
- Agent corrections get promoted to permanent rules, not repeated weekly
- Config changes don't cause mystery failures
- New sessions pick up where the last one left off
- You check docs before assuming, not after breaking

## Customization

These guidelines are designed to merge with project-specific instructions. Add them to your existing `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` or use them standalone.

For project-specific rules, add sections like:

```markdown
## Project-Specific

- All agents use Indonesian by default
- Finance agent handles budgeting only — no cross-domain tasks
- Cron jobs route through the orchestrator for cross-channel delivery
```

## Tradeoff Note

These guidelines bias toward **durability and consistency over speed**. For quick one-off tasks, use judgment — not every change needs the full rigor.

The goal is preventing drift and degradation on systems you run for months, not slowing down simple tasks.

## Credits

Inspired by [andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) by forrestchang.

## License

MIT
