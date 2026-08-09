# Implementer brief template

Write one brief file per issue into the run directory and prompt the implementer with its path. Start from the minimal brief:

> Implement issue #{N} (part of spec #{M}). Branch `{branch}` off `{base}` — treat the base as given. Use /tdd ($tdd in Codex) with pre-agreed seams where applicable. All repo gates green when finished, commit, then wait for review findings. Never push, rebase, or open a PR on your own — the orchestrator sends those instructions explicitly.

Every brief carries the task and the orchestration contract. The other sections state their own triggers; add one only when its trigger holds. Padding dilutes — the brief is the implementer's whole world.

```markdown
# Task: {issue title} (#{issue number})

Branch `{branch}`, based on `{base branch}` ({why this base: main | PR #N of this stack}). Treat the base branch's content as given.

## The task

{Only when the ticket alone leaves scope or prior decisions unclear: 2–6 sentences — what to build, the decisions already made and where (issue/spec links), what is explicitly out of scope.}

## Overlap warnings

{Only when another live branch or a recently merged PR touches the same files: name it, state which side is authoritative for what, and what must stay identical so rebases dedupe.}

## Pre-agreed testing seams

{Only when the ticket names no observable surfaces to test against: numbered seams, each naming the surface and the demand — e.g. "the bill of X: pin the product CODE and price, not the display label". Pin identities, not labels: a test that asserts a label passes while the wrong product is bought.}

## Orchestration contract

(Always included.)

- Implement with /tdd at the seams above. Do not invent seams; do not manufacture tests for pure renames.
- All repo gates green before you declare done.
- Never push, rebase, or open a PR on your own — the orchestrator owns integration and sends those instructions explicitly. A reviewer will examine your committed diff in rounds; each round you judge every finding FIX or REFUTE, writing one entry per finding with a reason to the disposition file the orchestrator names, fix what is real, re-run gates, commit, and wait.
- A question the spec does not answer gets an educated call: pick the answer you would defend (a second opinion from another agent is fair input), record the call + reasoning + reversal path in `{decisions file path}`, and continue. These entries become the PR body section "Decisions made without the owner" when the PR is opened — no open-question sections; state decisions.
- When the review converges: stop with a clean worktree and wait. The orchestrator will walk you through rebase, push, and PR creation, including the PR base and body.

## Repo-specific additions

{Only lines whose facts the implementer cannot discover in the repo:}

- First: {install command}, then read the repo's agent docs: {list}.
- Gate commands and caveats: {exact commands — e.g. "never bare `bun test`"}.
- Commit in repo style{; end commit messages with: {trailer}}.
```
