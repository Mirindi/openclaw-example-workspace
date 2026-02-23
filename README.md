# OpenClaw Agent Workspace — Example Configuration

This is a sanitized version of a real OpenClaw agent workspace that manages multiple production codebases, handles GitHub issues autonomously, and maintains persistent memory across sessions.

**What this repo is:** A reference for how to structure an AI agent's workspace so it actually remembers things, learns from mistakes, and works proactively.

**What this repo is NOT:** A plug-and-play template. You'll need to adapt these files to your own workflow, projects, and personality preferences.

## How It Works

The agent (running on Claude Opus via OpenClaw) reads these files at the start of every session:

1. `AGENTS.md` — Boot sequence + rules of engagement
2. `SOUL.md` — Personality, voice, working style
3. `IDENTITY.md` — Who the agent is
4. `USER.md` — Who it's helping (you)
5. `HEARTBEAT.md` — What to do during periodic check-ins
6. `learnings/LEARNINGS.md` — Rules from past mistakes
7. `WIP.md` — Current work in progress
8. `memory/YYYY-MM-DD.md` — Recent daily logs
9. `MEMORY.md` — Long-term curated memory (main session only)

The key insight: **if it's not written to a file, it doesn't exist.** The agent has amnesia between sessions. These files ARE the memory.

## Results

Using this system across three sprints over ~8 days on a WhatsApp commerce platform:

- 62 GitHub issues closed
- 58+ PRs merged (all reviewed by CodeRabbit, all feedback addressed)
- Sprint 3: 27 issues in 75 minutes — security fixes, UX overhaul, type safety

## File Descriptions

| File | Purpose |
|------|---------|
| `AGENTS.md` | Boot sequence, memory rules, write discipline, handover protocol |
| `SOUL.md` | Agent personality, voice, what to never do |
| `IDENTITY.md` | Agent name and role |
| `USER.md` | Owner profile (sanitized — yours would have real details) |
| `TOOLS.md` | Tool usage notes, CLI references, workarounds |
| `HEARTBEAT.md` | Periodic check-in behavior and proactive tasks |
| `WIP.md` | Current tasks, blockers, next steps |
| `MEMORY.md` | Curated long-term memory |
| `learnings/LEARNINGS.md` | One-liner rules from mistakes (this compounds!) |
| `memory/example-day.md` | Example daily log |
| `openclaw-config-example.json` | Sanitized OpenClaw config with comments |

## Key Concepts

### Write Discipline
After every task, the agent logs:
1. Decision + outcome → daily log
2. Mistakes → `learnings/LEARNINGS.md`
3. Significant context → flagged for MEMORY.md review
4. Current state → `WIP.md`

### Handover Protocol
Before session end or model switch, the agent dumps:
- What was discussed
- What was decided
- Pending tasks with exact details
- Next steps remaining

### Heartbeat System
Every ~30 minutes, the agent proactively:
- Checks for stale PRs or blocked work
- Runs self-updates
- Reviews memory and consolidates insights
- Backs up the workspace

### LEARNINGS.md (Most Underrated File)
Every mistake becomes a one-line rule. These compound over weeks into a personal operations manual built from the agent's own failures.

## Setup

1. Install [OpenClaw](https://openclaw.ai)
2. Copy these files to `~/.openclaw/workspace/`
3. Edit `USER.md` with your details
4. Edit `SOUL.md` with your preferred agent personality
5. Edit `TOOLS.md` with your actual tools and CLI references
6. Start chatting — the agent will evolve the files over time

## Memory Config (in openclaw.json)

The config enables:
- **Memory flush before compaction** — auto-saves important context to disk before the context window gets compressed
- **Hybrid search** (BM25 + vectors) — finds both exact matches and semantic matches
- **Temporal decay** — recent notes rank higher than stale ones
- **MMR re-ranking** — reduces duplicate snippets in search results
- **Embedding cache** — avoids re-embedding unchanged text

See `openclaw-config-example.json` for the full config structure.

## License

MIT — do whatever you want with it.
