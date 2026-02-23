# LEARNINGS.md — Rules From Mistakes

<!-- 
  One-line rules that compound over time.
  Every mistake becomes a rule so future-you never repeats it.
  Append-only during work. Review during heartbeats.
-->

## Agent Operations
- Never "mental note" anything. If it matters, write it to a file immediately.
- Never finish a conversation without updating memory/YYYY-MM-DD.md and WIP.md. Credits can die mid-sentence.
- Don't use `&` with `background:true` in exec — it backgrounds in the shell and exits PTY immediately.
- Claude Code agents die with code 143 (SIGTERM) on rate limits — re-launch with existing progress works.
- After 2-3 rounds of fixing code review issues, merge — remaining comments are refinements.
- Merge conflicts compound fast when 5+ PRs touch same files. Merge in sequence, not parallel.
- Always use `--dangerously-skip-permissions` for Claude Code agents. Without it they hang.
- Always set git config email before committing in worktrees.
- Always work in /tmp/ worktrees for PR work. Never clone to random directories.

## Technical
- Module-level caches (Set/Map) persist across test files in vitest. Export a reset function.
- [Add your own as you discover them]

## Communication
- Never read MEMORY.md in group chats or shared channels — security risk.
- Don't repeat info your human already knows. Keep it brief.
- When told "figure it out" — that's the cue to solve it, not ask more questions.
