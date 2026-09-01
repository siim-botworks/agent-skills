# Implementer brief template

Write one brief file per issue into the run directory and prompt the implementer with its path. Start from the minimal brief:

> Implement issue #{N} (part of spec #{M}). Branch `{branch}` off `{base}`; treat the base as given. Use /tdd ($tdd in Codex) against the surfaces the ticket names. Subagents are encouraged for exploration and second opinions. All repo gates green when finished, commit, stop with a clean worktree, and wait. Never push or open a PR on your own, and merge or rebase only when the orchestrator explicitly instructs it; the orchestrator owns integration.

The ticket and spec are the single source of truth for scope and testable surfaces; when a ticket is unclear or names nothing testable, fix the ticket (a comment is enough) before launch instead of restating scope in the brief, so implementer and reviewer read the same words. The brief carries only what the tracker and the repo cannot: run state and the orchestration contract. Repo facts (gates, install, commit style) live in the repo's agent docs, which the harness loads on its own. Padding dilutes; the brief is the implementer's whole world.

```markdown
# Task: {issue title} (#{issue number})

Branch `{branch}`, based on `{base branch}` ({why this base: default branch | the run's integration branch | PR #N of this stack}). Treat the base branch's content as given.

## Overlap warnings

{Only when another live branch or a recently merged PR touches the same files: name it, state which side is authoritative for what, and what must stay identical so merges dedupe.}

## Orchestration contract

(Always included.)

- Implement with /tdd against the surfaces the ticket names. Do not invent seams; do not manufacture tests for pure renames. Pin identities, not labels: a test that asserts a display label passes while the wrong product is bought.
- Subagents are explicitly authorized and encouraged: exploration, second opinions, parallel legwork.
- All repo gates green before you declare done.
- Never push or open a PR on your own, and merge or rebase only when the orchestrator explicitly instructs it. The orchestrator owns integration and runs it directly; you will be called back only for merge or rebase conflicts, integration gate failures, review findings, or new work.
- If review findings arrive, judge every finding FIX or REFUTE: one entry per finding with a written reason, in the disposition file the orchestrator names. Fix what is real, re-run gates, commit, and wait.
- A question the spec does not answer gets an educated call: pick the answer you would defend (a second opinion from another agent is fair input), record the call + reasoning + reversal path in `{decisions file path}`, and continue. These entries become the PR body section "Decisions made without the owner"; no open-question sections, state decisions.
- When your work is done: stop with a clean worktree and wait.

```
