# Implementer brief template

Write one brief file per issue into the run directory and prompt the implementer with its path. Start from the minimal brief:

> Implement issue #{N} (part of spec #{M}). Branch `{branch}` off `{base}`; treat the base as given. Use /tdd ($tdd in Codex) with pre-agreed seams where applicable. Subagents are encouraged for exploration and second opinions. All repo gates green when finished, commit, stop with a clean worktree, and wait. Never push, merge, rebase, or open a PR on your own; the orchestrator owns integration.

Every brief carries the task and the orchestration contract. The other sections state their own triggers; add one only when its trigger holds. Padding dilutes; the brief is the implementer's whole world.

```markdown
# Task: {issue title} (#{issue number})

Branch `{branch}`, based on `{base branch}` ({why this base: default branch | the run's integration branch | PR #N of this stack}). Treat the base branch's content as given.

## The task

{Only when the ticket alone leaves scope or prior decisions unclear: 2-6 sentences: what to build, the decisions already made and where (issue/spec links), what is explicitly out of scope.}

## Overlap warnings

{Only when another live branch or a recently merged PR touches the same files: name it, state which side is authoritative for what, and what must stay identical so merges dedupe.}

## Pre-agreed testing seams

{Only when the ticket names no observable surfaces to test against: numbered seams, each naming the surface and the demand, e.g. "the bill of X: pin the product CODE and price, not the display label". Pin identities, not labels: a test that asserts a label passes while the wrong product is bought.}

## Orchestration contract

(Always included.)

- Implement with /tdd at the seams above. Do not invent seams; do not manufacture tests for pure renames.
- Subagents are explicitly authorized and encouraged: exploration, second opinions, parallel legwork.
- All repo gates green before you declare done.
- Never push, merge, rebase, or open a PR on your own. The orchestrator owns integration and runs it directly; you will be called back only for merge conflicts, review findings, or new work.
- If review findings arrive, judge every finding FIX or REFUTE: one entry per finding with a written reason, in the disposition file the orchestrator names. Fix what is real, re-run gates, commit, and wait.
- A question the spec does not answer gets an educated call: pick the answer you would defend (a second opinion from another agent is fair input), record the call + reasoning + reversal path in `{decisions file path}`, and continue. These entries become the PR body section "Decisions made without the owner"; no open-question sections, state decisions.
- When your work is done: stop with a clean worktree and wait.

## Repo-specific additions

{Only lines whose facts the implementer cannot discover in the repo:}

- First: {install command}, then read the repo's agent docs: {list}.
- Gate commands and caveats: {exact commands, e.g. "never bare `bun test`"}.
- Commit in repo style{; end commit messages with: {trailer}}.
```
