# Orchestration rules on top of the herdr skill

Policy layered on the installed `herdr` skill — load that skill first; it and the live CLI own command syntax, lifecycle states, waiting, reading, and pane control. This file owns only what they do not: run artifacts, reviewer isolation, approvals, and harness quirks. When anything here disagrees with the live CLI or harness, follow what you observe, note the discrepancy in the report, and propose the file edit to the human — a run does not modify the skill package.

## Durable artifacts

Transcripts scroll off and compact away; the run-directory files (see the skill) are what arbitration and resumption stand on. When a pane's output is unrecoverable, ask the agent to write its answer to a run-directory file and record the path in `state.md`.

## Harness quirks

- Claude implementer args after `--`: `--model <model> --effort <level> --permission-mode acceptEdits`. If a start is blocked by a permission classifier, retry with a milder permission flag.
- Codex args after `--`: `-c model_reasoning_effort=<level> -c approval_policy=never` (Codex has no effort flag; the `-c` config is the way).
- Codex invokes skills with a `$` prefix (`$code-review`, `$tdd`), not `/`. The convention has changed before — if `$` misfires, read the pane for what Codex currently accepts.
- Prompt long briefs as a file path the agent reads, not inline text.

## Timeouts are not transitions

A wait or a prompt's `--wait` timing out on long work is normal, not failure. Check the agent's live state; re-arm the wait while it is `working`. An agent mid-compaction also reports `working` — wait for idle before prompting, or the prompt queues behind the compaction. Checkpoint every observed transition in `state.md`.

## Fresh reviewer per round

A reviewer pane may be reused; the reviewer *session* may not. Start a new conversation via the harness's own reset (in Codex, `/new`), confirm from the pane that the context actually reset, then send the round brief — contents per the skill's Fresh-reviewer rule, plus the report-file path to write. Renaming the agent afterward is bookkeeping for your state table, not evidence of freshness.

## Blocked panes (approval policy)

When an agent goes `blocked`, read the exact visible prompt before acting — never assume which key means what:

- Reads of run-directory files, ordinary dev commands, test runs inside the worktree: approve, preferring a "don't ask again this session" option when the prompt offers one.
- Destructive or outward-facing actions (deletes outside the worktree, pushes not yet earned by a join instruction, network posts): leave blocked, log it, surface it to the human or make an executive call per the skill.
