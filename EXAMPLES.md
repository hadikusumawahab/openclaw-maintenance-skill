# Examples

Real-world OpenClaw scenarios demonstrating the five principles. Each example shows what commonly goes wrong and how to fix it.

---

## 1. Lean Workspace

### Bloated Instruction Files

**Problem:** AGENTS.md has grown to 15K+ characters over months of additions. Compaction triggers after a few messages, and the agent loses track of what matters.

**Bad:**

```markdown
# AGENTS.md — 400+ lines

## Session Startup
[50 lines of startup checklist]

## Communication Rules
[80 lines of messaging rules, many redundant]

## Tool Usage
[100 lines of tool-specific instructions, half outdated]

## Memory Rules
[60 lines of memory management, overlapping with TOOLS.md]

## Reminder System
[50 lines duplicated from another agent's AGENTS.md]

## Historical Notes
[60 lines of past decisions that should be in memory files]
```

**Good:**

```markdown
# AGENTS.md — ~150 lines, organized by topic

## Session Startup
[10 lines: read these files, check memory, greet]

## Agent-Specific: [Role]
[40 lines: unique responsibilities, delegation rules]

---

# Generic (shared across agents)

## Memory Rules
[20 lines: when to update, what to prune]

## Communication
[20 lines: language, format, channel rules]

## Tools
[15 lines: preferences, not full documentation]
```

**Why:** AGENTS.md is injected at session start and counts against the 20K char-per-file limit. Files that exceed this are silently truncated — rules at the bottom disappear. A focused file stays under the limit and leaves room for actual conversation. Move tool docs to TOOLS.md, old decisions to memory files, repeated patterns to skills.

---

## 2. Skills Over Bloat

### Duplicated Instructions Across Agents

**Problem:** Multiple agents each have the same web search instructions copy-pasted into their TOOLS.md. When the workflow changes, you update one and forget the others.

**Bad:**

```
workspace-agent-a/TOOLS.md:  "To search the web, use tavily..."  (30 lines)
workspace-agent-b/TOOLS.md:  "To search the web, use tavily..."  (30 lines, slightly different)
workspace-agent-c/TOOLS.md:  "To search the web, use tavily..."  (30 lines, outdated version)
workspace-agent-d/TOOLS.md:  "To search the web, use tavily..."  (30 lines)
```

**Good:**

```
workspace/skills/tavily/
├── SKILL.md            # Metadata + usage docs
├── tavily_search.py    # Single implementation
└── .venv/              # Isolated dependencies

# SKILL.md
---
name: tavily
description: Web search and content extraction
metadata: {"clawdbot":{"requires":{"bins":["python3"],"env":["TAVILY_API_KEY"]}}}
---

# Usage
tavily_search.py search "query" --max-results 5
```

**Why:** One skill, one location, all agents can invoke it. Only ~100 tokens of metadata in context per agent instead of 30 lines each. Update once, applies everywhere.

---

## 3. Design for Continuity

### Decisions Lost in Chat

**Problem:** You decided to change the default model during a Tuesday chat. By Friday, a new session has no memory of this — the agent uses the old model again. You re-explain. Again next week.

**Bad:**

```
# Tuesday chat (not persisted anywhere)
User: "Switch default model to kimi-k2.5 from now on"
Agent: "OK, updated openclaw.json"

# Friday chat (new session, no context)
User: "Why is the agent still using the old model?"
Agent: "Let me check... I don't see any record of a model change decision"
```

**Good:**

```markdown
# In MEMORY.md (persisted immediately after decision)
## Pending Decisions
- Model migration: switched default to kimi-k2.5 on 2026-04-08.
  Reason: better quality, lower latency. Review after 2 weeks.

# In AGENTS.md (if it becomes a permanent rule)
## Model Preferences
- Default: kimi-k2.5. Previous model as fallback only.
```

**Why:** Your instruction files should tell agents: "When user makes a key decision, persist it to MEMORY.md immediately. If it's permanent, promote to AGENTS.md." Decisions in chat die with the session.

---

## 4. Trust but Verify

### Outdated Config Fields After an Update

**Problem:** After updating OpenClaw, you ask the AI to add a new channel binding. It uses field names from its training data — but those fields were renamed two versions ago. OpenClaw starts fine, no errors, but the binding silently does nothing. You spend hours debugging before realizing the config structure changed.

**Bad:**

```
User: "Add a Telegram binding for my project agent"

Agent: "I'll add this to openclaw.json:"
{
  "bindings": [
    { "agent": "project-bot", "channel": "telegram", "chatId": "123456789" }
  ]
}
// Field names are outdated — no error, just silently ignored

# You also skip validation
$ openclaw restart
# Agent never responds to Telegram. Hours of debugging.
```

**Good:**

```
User: "Add a Telegram binding for my project agent"

Agent: "Let me check the current docs first..."
[Fetches docs.openclaw.ai/llms.txt or /gateway/configuration]

"Based on the current config reference:"
{
  "agents": {
    "bindings": [
      { "agentId": "project-bot", "channelId": "telegram-project", "match": { "peerId": "123456789" } }
    ]
  }
}

# After applying, always validate
$ openclaw doctor
# Confirms config is valid, binding is recognized
```

**Why:** OpenClaw ships frequent updates. Config fields get renamed, CLI flags change, features behave differently between versions. The two habits that prevent silent failures: fetch current docs before making changes, and run `openclaw doctor` after every change. Fetching docs takes 5 seconds; debugging a silent config failure takes hours.

---

## 5. Design for Agent Coherence

### Duplicate Skills With Different Names

**Problem:** You ask the assistant to add a web search capability to Agent A. It creates a new skill. But Agent B already has a web search skill with a different name — now you have two skills doing the same thing, maintained separately, slowly diverging.

**Bad:**

```
User: "Add web search to Agent A"

Assistant creates:
workspace/skills/web-search/
├── SKILL.md
└── search.py

# Meanwhile, already exists but assistant didn't check:
workspace/skills/tavily/
├── SKILL.md
└── tavily_search.py
# Same functionality, different name. Now maintained in two places.
```

**Good:**

```
User: "Add web search to Agent A"

Assistant checks existing skills first:
$ ls workspace/skills/
tavily/  yt-summary/  workflowy-todo/  ...

"Agent B already has a web search skill at workspace/skills/tavily/.
I'll make this available to Agent A as well — no need to create a new one."
```

**Why:** Without system awareness, the assistant treats each agent as isolated. Over time this creates duplicate skills, conflicting instructions, and maintenance overhead that grows with every agent. Check what exists across the system before adding anything new.

---

## Anti-Patterns Summary

| Principle | Anti-Pattern | Fix |
|-----------|-------------|-----|
| Lean Workspace | AGENTS.md hitting 20K char limit, rules silently truncated | Check `wc -m`, migrate content to skills/memory |
| Skills Over Bloat | Same instructions copy-pasted across agents | One shared skill, metadata-only in context |
| Design for Continuity | Decisions exist only in chat history | Persist to MEMORY.md immediately, promote to AGENTS.md if permanent |
| Trust but Verify | Assume config fields haven't changed after update | Fetch docs before changes, run `openclaw doctor` after |
| Design for Agent Coherence | Add skill to one agent without checking others | Check existing skills and agent scopes before creating anything new |

## Key Insight

The difference between agents that degrade over time and agents that stay sharp is **maintenance discipline**. These aren't one-time setup steps — they're ongoing practices. Workspace files need pruning, memory needs curation, config needs validation, docs need checking, and the system needs coherence. The agents that work best six months from now are the ones you maintain today.