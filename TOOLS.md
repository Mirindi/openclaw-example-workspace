# TOOLS.md - Technical Playbook

**Rule #1: Check this file BEFORE saying "I can't do that." You probably can.**

## Channels
<!-- List your configured messaging channels -->
- **Slack**: [workspace name]
- **Telegram**: Enabled
- **Webchat**: Active

## CLI Tools
<!-- List tools your agent has access to -->
- **gh** (GitHub CLI): Installed, authenticated
- **claude** (Claude Code CLI): Installed — see coding agent rules below

## Things That Don't Work (and what to do instead)

<!-- 
  This section is GOLD. Every time you hit a wall, document the workaround.
  Your agent will check this before giving up.
-->

### Example: Claude Code Rate Limits
Claude Code CLI has hourly rate limits. When hit, agents die.
- **Don't launch 4+ agents simultaneously** — spread them out
- **If agents die from rate limits:** wait for reset, then relaunch

### Example: Claude Code Permissions
**Always use `--dangerously-skip-permissions`** when spawning coding agents. Without it, agents hang waiting for permission prompts.

## Coding Agent Rules

### Repo Hygiene
1. Always use git worktrees in `/tmp/` for PR work. Never clone to random dirs.
2. After merging a PR, remove the worktree.
3. Canonical repos live in their designated directories.

### PR Workflow
1. Create feature branch from main
2. Make changes, commit with conventional commits
3. Push and open PR
4. Wait for code review (CodeRabbit, etc.)
5. Address ALL comments
6. Merge when clean (squash merge)
7. Monitor CI/CD — if deployment fails, fix immediately

## Setup Needed
<!-- Track what still needs configuring -->
- [ ] TTS voice preference
- [ ] Google Workspace auth
- [ ] [Other tools you plan to add]
