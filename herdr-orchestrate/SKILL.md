---
name: herdr-orchestrate
description: Orchestrate a multi-worktree implement-and-review run inside Herdr — one implementer agent per issue, fresh-reviewer loops to convergence, and (for a spec with sub-issues) one stacked-PR spine. Argument is a parent spec issue, or a list of independent issues.
disable-model-invocation: true
---

You are the **orchestrator**: you chart the work, launch and arbitrate the agents, and deliver the report. You write no feature code yourself — the join steps too are executed by the issue's implementer agent under your instruction.

## The loop rules

Every issue runs an implement-and-review loop under these rules:

- **Fixed point**: immediately after creating the worktree, record the base **SHA**, not just the branch name. Every review round diffs `fixed-point...HEAD`, so each round's reviewer sees the full change, not just the latest fixes.
- **Fresh reviewer**: every round's reviewer is a new harness conversation with no task history. A reused pane is fine only after a verified context reset (see HERDR-OPS.md). Its brief is compact, never conversation history: the ticket reference (and parent spec), the change scope, gates already green, and from round 2 on the earlier rounds' fixed/refuted list with reasons, so it hunts new issues instead of relitigating. Findings only, no code changes, numbered, severity-ordered, file:line references, written to `review-r{K}.md` in the run directory — or the exact sentence "No findings."
- **Fix/refute**: the implementer judges every finding — reviewer output is advice, not authority. It writes `disposition-r{K}.md` with one FIX or REFUTE entry per finding: fix what is real; refute what is wrong, already handled, out of scope, or a deliberate tradeoff, with a written reason. A high-severity correctness or security finding is refuted only after checking the code path it names. Re-run gates after fixes, then commit the round.
- **Converged**: a round changes no code — the worktree is clean and either the reviewer wrote "No findings.", or every finding has a REFUTE entry and HEAD equals the round-start HEAD. Any FIX means another round. You arbitrate from the two files and the SHAs, not from transcript memory: every finding has exactly one disposition, and the clean-worktree + HEAD comparison decides.
- **Cap**: 8 rounds. A capped issue stops and is reported, not silently shipped. Join reviews (below) are a separate epoch: same rules, cap 2, own file names (`review-join-r{K}.md`, `disposition-join-r{K}.md`) so implementation-round files stay immutable.

## Guard

Run `test "${HERDR_ENV:-}" = 1`. If it fails, say you are not inside Herdr and stop.

## Preflight

Discover, then propose from what exists:

1. Harnesses: which of `claude` and `codex` are installed and logged in.
2. `gh` authenticated; `gh stack` extension installed (spine mode needs it).
3. The `herdr` skill installed — load it before issuing Herdr control commands; it and the live CLI own command syntax. Then read [`HERDR-OPS.md`](HERDR-OPS.md) for this run's extra rules — before the first launch, not after something misbehaves.
4. The repo's gates: its lint/typecheck/format/test commands and any test-runner caveats, from package scripts and the repo's agent docs.
5. Read [`MODELS.md`](MODELS.md) for model choice; propose only models the installed harnesses actually offer.

## Chart

Read the argument. The invocation usually states most of the plan — apply what it states and chart only what it leaves open:

- **Parent spec issue** with sub-issues → **spine mode** by default: the run produces one stacked-PR chain.
- **List of issues** → investigate dependencies among them anyway: native blocking relations, statements in the bodies, and overlap in the code they will touch. Then propose spine mode or **independent mode** (each converged branch PRs straight to the default branch; no joining). A stack without hard dependencies is still legitimate when the user wants one reviewable train — the pause decides.
- Anything unusual in the invocation → carry your interpretation and the unresolved choice into the approval pause; ask earlier only when no safe plan can be charted without the answer.

Build the dependency DAG from the tracker's **native blocking relations**. If issue bodies also carry a dependency convention, compare; report any disagreement at the pause instead of silently picking a side.

Then draft the run plan:

- **Spine**: one topological order of all issues — the bottom-to-top PR order. Prefer fronting any externally time-critical track (a demo, a deadline named in the spec).
- **Wave width**: how many worktrees build concurrently (2–3 typical), or fully sequential. Parallelism is a throughput dial, not a structure — the spine is linear either way.
- **Cast**: per issue, implementer model + effort; one reviewer model + effort for the run. Choose per MODELS.md and say why for each non-default pick.
- **Permission posture**: default auto/accept-edits with orchestrator-handled approvals; bypass only if the user says this machine is disposable.

## The pause

Present the whole plan — mode, spine, width, cast, posture, any DAG disagreements — as one message and **wait for approval**. Create nothing before it. Record approved overrides exactly; if an override conflicts with the guard, repo safety rules, or an unavailable capability, state the conflict and pause again instead of silently approximating.

After approval, create the **run directory** in your scratchpad and tell the user its absolute path. It holds `state.md` plus every brief, review, disposition, and decisions file. `state.md` records: the approved plan (mode, spine, DAG, width, cast, posture), repo/remote/default branch, stack identity once created, and per issue: implementation fixed-point SHA, join-review fixed point and round once a join epoch starts, worktree, agent and pane ids, branch, phase, round, artifact paths, gate results, PR number, educated calls, human interventions, and next action. Phases: PLANNED, IMPLEMENTING, REVIEWING, CONVERGED, JOINING, JOINED, OPEN, CAPPED, BLOCKED.

Update `state.md` on every state change. **After compaction, restart, or a human pane intervention**: re-read this skill and `state.md`, reconcile against live Herdr agent state, `git` HEADs, and GitHub PR/stack state — the file is a checkpoint, not authority over newer live state — record any discrepancy, then resume from the next-action column.

## Run

The approved wave width is the scheduler bound: at most that many issues IMPLEMENTING or REVIEWING at once — start the next runnable issue only when a slot opens. Joining stays serialized in spine order regardless.

An issue becomes runnable when its implementation base is unambiguous: roots and independent mode → the default branch; one blocker → that blocker's converged head; multiple blockers → wait until all of them have **joined**, then base on the current stack tip. Never pick one of several blockers arbitrarily. Then:

1. `herdr worktree create` off that base — never the spine position — and record the fixed-point SHA.
2. Start the implementer in the worktree pane, in its native harness with the agreed effort.
3. Prompt it with a brief built from [`BRIEF-TEMPLATE.md`](BRIEF-TEMPLATE.md) — a file path it reads, not a wall of pasted text.
4. Before round 1, verify the implementation is committed, the worktree clean, and the gates green. Then loop until converged or capped, per the loop rules: record round-start HEAD → split a sibling pane and start a **fresh** reviewer → reviewer writes `review-r{K}.md` → hand the implementer the path with "judge fix/refute into `disposition-r{K}.md`, gates, commit" → arbitrate from the two files and the HEADs → checkpoint `state.md` → repeat.

**The human may jump into any pane.** A human turn in a transcript is authoritative: resync from the pane's current state, never prompt over a live conversation, and log the intervention in the state file.

## Join (spine mode)

A converged branch **joins** the stack only when everything below it in the spine has joined. You instruct the issue's implementer through the steps, in order:

1. Rebase only the issue's own commits: `git rebase --onto <stack-tip> <fixed-point> <branch>` — never let Git infer the replay boundary, or it can replay the dependency's commits.
2. Full gates on the rebased head.
3. A conflicted rebase is new hand-written code: run one join-review epoch (fresh reviewer, join-review fixed point = the new base, cap 2) on the rebased diff. The implementation fixed point stays untouched in `state.md` — it remains the `--onto` replay boundary. A clean rebase joins on gates alone.
4. Push `--force-with-lease`; open the PR **based on the previous stack-tip branch** (default branch for the bottom PR); `gh stack link` it (the `gh-stack` skill, if installed, owns the mechanics). Verify from `gh` output that the remote head equals the validated SHA, the PR base is the immediate lower branch, and stack membership and order match the approved spine — then record JOINED; the tip advances. Publication happens only here: no implementer pushes or opens a PR outside a join instruction.

## Publish (independent mode)

A converged branch becomes OPEN through the same implementer-under-instruction pattern: refresh the default branch and, if it moved, `git rebase --onto <default-tip> <fixed-point> <branch>`; full gates (a conflicted rebase enters a join-review epoch first); push; open the PR against the default branch; verify remote head and PR base from `gh` output; record the PR and validated SHA, then OPEN.

## Stuck issues

A capped or otherwise stuck issue does not block the train silently: it and **every issue reachable from it in the DAG** are excluded; an unrelated later issue may still join via its own `--onto` replay, which by construction carries none of the failed issue's commits. When exclusions break the spine, stop at the highest joinable prefix and say so. Leave stuck worktrees intact.

## Educated calls

Mid-run questions the spec did not answer get an **educated call**, not a stall: the implementer picks the answer it would defend (a second opinion from another agent is a legitimate input), records the call, its reasoning, and its reversal path in the issue's decisions file in the run directory, and continues. The calls go into the PR body ("Decisions made without the owner") when the PR is opened, and the report aggregates them so the human reviews them in one place. Only a fork that is expensive to reverse in both directions stops an issue — then report it as blocked.

You hold the same license: when an implementer wedges itself — a stalled judgment, circling on a refuted finding, a blocked approval it cannot see past — make the executive call that unblocks it, log it in the state file, and include it in the report.

## Report

The run **terminates** when every planned issue is in a terminal phase — JOINED, OPEN (independent mode), CAPPED, or BLOCKED — with its evidence in `state.md` (a capped entry names the unresolved findings; a blocked entry names the exact blocker and next recovery action), and the report is delivered. The run **succeeded** only if every issue is JOINED or OPEN — say which of the two you are reporting.

The report lists: PRs in spine order, rounds per issue with fixed/refuted counts, every educated and executive call, every human intervention, everything capped or blocked with its downstream exclusions, and the recommended merge order.
