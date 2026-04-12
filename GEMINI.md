# OpenClaw Maintenance Guidelines

Five principles for keeping your OpenClaw agents lean, consistent, and reliable over time.

**Tradeoff:** These guidelines bias toward durability and consistency over speed. For quick one-off tasks, use judgment.

## 1. Lean Workspace

**Every line in a workspace file costs tokens every turn. Earn its place.**

- Agent behavior is driven by workspace markdown files (SOUL.md, AGENTS.md, TOOLS.md), not code. When the user wants to change how an agent behaves, edit its workspace files — not a config or script.
- OpenClaw enforces hard limits: 20,000 chars per file (`bootstrapMaxChars`) and 150,000 chars total across all workspace files (`bootstrapTotalMaxChars`). Files that hit the limit are silently truncated — rules at the bottom of the file disappear without warning.
- Check workspace health: run `wc -m <file>` to see character counts. Under 50% of limit = safe. Over 75% = migrate content to skills or memory files before you lose rules to truncation.
- Touch only what you must when editing — surgical edits over full rewrites; match existing style and structure.
- When a file is getting bloated, identify what can move: tool documentation → TOOLS.md, historical decisions → memory files, repeated workflows → skills.

Ask yourself: "Would the diff be clean and reviewable?" If not, you're changing too much.

## 2. Skills Over Bloat

**Extract repeating patterns into modular skills, not instruction sprawl.**

- When you notice the same instructions duplicated across multiple agents, extract them into a shared skill instead of maintaining copies in each workspace.
- Skills use progressive disclosure: only metadata (~100 tokens) is in context; full instructions load when the agent reads the skill file — massive token savings over putting everything in workspace files.
- One canonical location for shared skills; never duplicate across agent workspaces. When updating a workflow, update the skill once — it applies everywhere.
- If a user asks to add a workflow that already exists as a skill, point them to the existing skill instead of adding instructions to AGENTS.md or TOOLS.md.
- When building new workflows, suggest creating a skill if it's likely to be reused across agents or sessions.

Ask yourself: "Is this instruction reusable across agents or sessions?" If yes, it's a skill. If no, it belongs in agent instructions.

## 3. Design for Continuity

**Agents forget everything between sessions. If it's not in a file, it didn't happen.**

When agents keep forgetting things or repeating mistakes, the problem is almost always that important context lives in chat instead of files. Here's how to diagnose and fix it:

- If an agent "forgot" something — check if it was persisted to a file. Forgotten decisions go to MEMORY.md now; repeated corrections should become permanent rules in AGENTS.md, not chat instructions that die with the session.
- Keep MEMORY.md curated: prune completed tasks, archive old decisions to topic files. Layer the `memory/` directory by durability — working memory (curated) for current state, daily logs for temporal context, topic files for persistent domain knowledge. Promote useful daily entries to permanent files, then clean up.
- Rules and learnings belong in files loaded at session start (AGENTS.md, TOOLS.md) — not in files the agent never reads back. Write self-learning triggers into AGENTS.md so the agent updates its own docs when corrected. This isn't built-in — you architect it.
- Don't rely solely on auto-compaction. Write important state to files proactively before conversations get long. Add a "search memory before acting" rule to AGENTS.md — without this, agents guess instead of checking their own notes.
- If dreaming is enabled, it handles memory consolidation automatically. Focus on what dreaming doesn't cover: promoting corrections to instruction files, cleaning up stale memory, and diagnosing continuity gaps.

## 4. Trust but Verify

**Config changes, CLI commands, and docs evolve. Trust your setup, but always verify it still works.**

- When unsure about a config field, feature, or best practice — fetch the docs first at https://docs.openclaw.ai/. OpenClaw evolves fast; don't rely on stale assumptions or LLM training data.
- Run validation and diagnostics after every config change — catch drift before it causes silent failures.
- Watch for wizard and onboard tools silently modifying config — they may reset tool access profiles or drop custom keys after OAuth refresh.
- Check the running version and match solutions to it — features and flags change between releases.
- Before recommending a feature, flag, or config field: verify it still exists in the current version.
- Diagnostic sequence when things break: check status → check gateway → check logs → run doctor/validator.

## 5. Design for Agent Coherence

**Before changing one agent, understand the whole system.**

- Never share state directories across agents — each agent needs isolated auth, sessions, and workspace to avoid collisions.
- Before modifying an agent, read its scope boundary explicitly — don't assume from the agent's name what it does or doesn't handle.
- Before adding a skill to one agent, check if an equivalent skill already exists on another agent — avoid duplicates with different names doing the same thing.
- If instructions should be identical across multiple agents, treat them as shared content — not copy-pasted sections that will inevitably diverge when edited separately.
- When editing instruction files, keep agent-specific sections clearly separated from generic/shared sections so syncing across agents doesn't accidentally overwrite unique rules.
- Before making a system-wide change, list all affected agents first. Work through them sequentially and verify consistency after finishing.
- When in doubt about agent scope or shared resources, check the workspace directory structure and any existing sync tooling before proceeding.
