---
name: implement-review-loop
description: Implement a ticket with TDD, then run fresh-reviewer code-review rounds until converged, and report what was fixed and what was refuted.
disable-model-invocation: true
---

Implement a ticket, then loop code review with fresh reviewers until **converged**: a review round returns no findings, or every remaining finding is refuted with a written reason.

The argument is the ticket: an issue tracker URL or ticket number, or a path to a local markdown file.

## Setup

1. Read the ticket. If anything needs clarification, ask before you start.
2. Ask the user where to work: the current checkout, a new branch, or a worktree.
3. Confirm the reviewer with the user: which model reviews, and at which reasoning effort. Default: `gpt-5.6-sol` at `high`. Prefer a dynamic Workflow for reviewers when the harness supports it — it sets model and effort per reviewer. Skip the effort question when the chosen mechanism cannot set effort; say so instead of asking.

**Do not proceed with implementation until the above mechanics have been settled.**

## Implement

Implement the ticket. Use the TDD skill where applicable. Run the project's validation (tests, lint, typecheck) and get it green, then commit — before the first review round.

## Review loop

Use the /code-review skill to run review rounds until converged. Each round:

1. Spawn one or more **fresh** reviewers — agents with no prior involvement; a reviewer from an earlier round is never reused. Give each a compact brief, not conversation history:
   - the ticket (or a 1–3 sentence summary),
   - the change scope (files/modules touched),
   - checks already passing,
   - from round 2 on: what earlier rounds found, what was fixed, and what was refuted and why — so reviewers look for new issues in the fixes and anything missed, instead of relitigating.
   - Ask for findings only, ordered by severity, with file:line references. Reviewers do not change code.
2. Judge every finding yourself; reviewer output is advice, not authority:
   - **Fix** findings that are real: bugs, broken contracts, missing tests around changed behavior, violations of project conventions.
   - **Refute** findings that are wrong, already handled, out of scope, or a deliberate tradeoff — and record the reason. Refute a high-severity correctness or security finding only after checking the code path it names.
3. Re-run validation after fixes, then commit the round's fixes (unless the user said not to commit).
4. If the round produced any fixes beyond the trivial, run another round of /code-review and repeat until no more actionable findings.

## Report

When converged, report to the user:

- What was implemented and how it was validated.
- Rounds run and the reviewer model/effort used.
- Findings fixed.
- Findings refuted, each with its reason.
- Anything still open, if the loop stopped without converging.
