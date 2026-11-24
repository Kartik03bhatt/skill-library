---
name: agent-checkpointing
description: Make long-running agents resumable, idempotent and safe to retry. Use when an agent task runs for many steps or hours, when crashes or timeouts lose progress, when retries cause duplicate side effects, or when a human must approve steps mid-run.
---

# Agent checkpointing

Long agent runs fail: rate limits, crashes, timeouts, a context reset, a human closing the laptop. Design so a failure costs one step, not the whole run.

## Principles

1. **State outside the model.** Progress lives in a file or database, not only in the conversation. The context can be lost or compacted; the checkpoint cannot.
2. **Small, named steps.** Each step has an ID, inputs, outputs and a status: pending, running, done, failed.
3. **Idempotent actions.** Running a step twice gives the same result as once. Use upserts, deterministic IDs and "create if absent".
4. **Write the checkpoint after the step, and the intent before it** for actions with side effects, so a crash in between can be detected and reconciled.
5. **Resume by reading state, not by replaying the chat.**

## Checkpoint contents

- Goal and constraints (verbatim)
- Plan with step statuses
- Outputs of finished steps, or paths to them
- Pointers to files, branches and external IDs created
- The next action
- Timestamps and a run ID

Keep it small enough to load in full at the start of every resumed run. This doubles as the handoff for compaction; see `context-compaction`.

## Side effects

- Reads are free to retry. Writes need an idempotency key.
- For irreversible actions (send email, charge, delete, publish), record intent, act, record result. On resume, if intent exists without a result, check the outside world before retrying.
- Prefer dry-run first, then apply, for batch operations.
- Never retry blindly on a timeout; the first call may have succeeded.

## Retries

- Retry only transient errors, with exponential backoff and jitter.
- Cap attempts per step and per run.
- After the cap, mark the step failed with the error and stop; do not loop.
- Distinguish "step failed" from "plan is wrong". Repeated failures on one step should trigger a replan, not more retries.

## Human in the loop

- Place approval gates before irreversible or costly steps, not after.
- Persist the pending approval with enough detail to decide without the original chat.
- A run waiting on a human should consume nothing and resume from the checkpoint when answered.

## By use case

- **Data migration:** checkpoint per batch with the last processed key; make writes upserts; verify counts at the end.
- **Coding task:** commit to a work branch at each green step; the commit log is the checkpoint.
- **Research report:** save each source's notes as it is read; draft sections to files.
- **Scheduled monitoring agent:** store last-seen state so each run diffs against the previous one and does not re-alert.
- **Multi-step customer workflow:** record each completed external action with its confirmation ID.

## Checklist

- [ ] State stored outside the context
- [ ] Every step idempotent or guarded by an intent record
- [ ] Retry cap and backoff set
- [ ] Resume path tested by killing a run mid-step
- [ ] Approval gates before irreversible actions
