---
name: review-with-subagents
description: Iterative independent code review using fresh sub-agents. Use when Codex has made or is about to finalize code changes and the user asks for fresh-agent review, sub-agent review, an iterative review loop, "keep reviewing until clean", or wants review context such as a GitHub issue number, task description, reasoning effort, or focus areas passed to reviewers.
---

Run a fresh sub-agent review loop over the current code changes, then assess and fix relevant findings before repeating. Stop only when a fresh reviewer reports no actionable findings, the remaining findings are intentionally rejected, or a practical limit is reached and disclosed.

## Inputs

Before reviewing, identify:

- Review context: issue number when the issue has enough detail; otherwise a concise task summary.
- Change scope: changed files or modules.
- Passing checks: validation commands already run and passing. Include failed checks only when they explain current risk or reviewer focus.
- Reviewer hints: optional reasoning effort and focus areas such as correctness, tests, data migrations, Effect patterns, security, performance, accessibility, or backwards compatibility.

If the user gives no hints, choose a conservative default:

- Reasoning effort: `medium` for narrow changes, `high` for cross-module behavior, migrations, security-sensitive work, or complex refactors.
- Focus areas: correctness, regressions, missing tests, edge cases, and consistency with existing project conventions.

## Preflight

Confirm that the current Codex session has an available sub-agent mechanism. Use that mechanism to create each reviewer as a fresh independent agent. If no sub-agent mechanism is available, disclose that the skill cannot run as designed and ask whether to continue with a weaker self-review or manual review instead.

## Workflow

1. Inspect the work locally enough to identify the issue number or write a concise task summary, changed scope, and already-passing checks.
2. Spawn one fresh sub-agent for review. Do not reuse an earlier reviewer for later passes.
3. Give the reviewer only the compact context it needs: issue number or short task summary, changed scope, passing checks, and focus hints. Ask for findings only, ordered by severity, with file and line references where possible.
4. While the reviewer runs, only do non-code work, validation, or inspection unless the code work is unrelated to the reviewed scope. Include any code changes made during review in the next fresh review pass.
5. When the reviewer returns, assess every finding. Treat sub-agent output as advice, not authority.
6. Fix findings that are relevant and materially improve the code. Reject findings only when they are wrong, already handled, outside scope, or the tradeoff is explicitly justified. Do not quietly reject high-severity correctness, security, data-loss, or migration findings; fix them or disclose the decision and ask the user when appropriate.
7. Run the smallest useful validation after fixes.
8. Spawn a new fresh sub-agent with updated context and a summary of accepted fixes and intentionally rejected findings.
9. Repeat until a fresh reviewer reports no actionable findings, or only rejected findings remain.
10. Before finishing, run the repository's required final validation commands. If full validation is impractical, disclose exactly what was skipped and why.

## Reviewer Prompt Shape

Use a prompt like this, adapting details to the repository:

```text
Review the current code changes as an independent reviewer.

Context:
- <issue number if sufficient, otherwise 1-3 sentence task summary>

Scope:
- <changed files or modules>

Checks already passing:
- <commands that passed; omit commands not run>

Focus most on:
- <correctness/tests/security/performance/etc.>

Do not rerun checks already listed as passing unless you have a specific reason. Return only actionable findings. For each finding include severity, file/line, why it matters, and a concrete suggested fix. If there are no findings, say so clearly. Do not make code changes.
```

For later passes, add:

```text
Prior review results:
- Accepted and fixed: <summary>
- Intentionally rejected: <summary with reason>

Please look for remaining issues, regressions introduced by the fixes, and findings not already rejected.
```

## Assessment Rules

- Prefer fixing findings that indicate real user-visible bugs, broken contracts, missing validation, weak tests around changed behavior, or violations of established local patterns.
- Do not churn code for stylistic preferences unless they align with explicit project conventions.
- Do not accept speculative findings without checking the code path.
- Do not reject high-severity findings without either fixing them or making the unresolved risk explicit to the user.
- Do not let repeated reviewers relitigate intentionally rejected findings unless new evidence appears.
- If the same class of issue appears repeatedly, broaden the local inspection before spawning the next reviewer.

## Stopping Criteria

Stop when one of these is true:

- A fresh reviewer reports no actionable findings.
- The only remaining findings have been checked and intentionally rejected.
- The review loop reaches three passes without converging. In that case, summarize remaining findings, what was fixed, what was rejected, and why stopping is practical.

## Final Response

Report:

- Review passes completed and reviewer reasoning effort used.
- Relevant findings fixed.
- Findings rejected, with short reasons.
- Validation commands run and any remaining risk.
