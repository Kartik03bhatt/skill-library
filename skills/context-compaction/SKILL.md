---
name: context-compaction
description: Shrink an agent's context when a long task nears the context limit, without losing the thread. Use when designing summarization or compaction for a long-running agent, deciding what to keep, drop or externalize, or when an agent gets confused or repetitive after compaction.
---

# Context compaction

Compaction replaces a long history with a shorter one the agent can keep working from. Done badly, the agent forgets decisions, redoes finished work, or loses the user's constraints. Done well, a task can run far past one window.

## Pick the lightest tool that works

Try in this order:

1. **Clear stale tool results.** Old file dumps, search output and command logs are the biggest and least valuable tokens. Keep the call, drop the output, leave a one-line stub ("read src/app.ts, 340 lines").
2. **Externalize.** Write notes, plans and findings to a file or memory, then drop them from context. Re-read on demand.
3. **Summarize.** Replace the oldest part of the conversation with a structured summary.
4. **Restart with a handoff.** Start a fresh context seeded with the summary and the files that matter.

Provider-side features for clearing tool results and automatic compaction exist; check current docs. They save you writing the plumbing but do not decide what matters.

## Trigger

Compact at a threshold, not at the wall. Around 60 to 75 percent of the window leaves room to write a good summary. Compacting at 98 percent produces a rushed one or fails.

## What a summary must contain

Use fixed sections so nothing is skipped:

- **Goal:** the user's request in their words, plus any changes to it.
- **Constraints and preferences:** things the user said not to do. These are lost first.
- **Done:** completed steps, with the result of each, not the process.
- **Decisions:** what was chosen and why, including options rejected.
- **State:** files changed, commands that worked, current branch, running processes.
- **Open questions and blockers.**
- **Next step:** the single next action.

Keep exact identifiers verbatim: paths, function names, IDs, error strings, URLs. Paraphrased identifiers cause hallucinated ones later.

## What to drop

- Raw tool output that is already reflected in a decision.
- Dead ends, unless the reason they failed is useful ("approach X fails because Y").
- Pleasantries and restated instructions.

## Interactions

- Compaction rewrites history, so it invalidates the prompt cache after the edit point. Compact rarely and in big steps rather than trimming a little every turn. See `prompt-caching`.
- Anything that must survive compaction belongs in memory or a file, not only in the chat. See `agent-memory-design`.

## By use case

- **Coding agent:** keep the plan, the list of changed files and failing tests. Drop file contents; they can be re-read.
- **Research agent:** keep claims with source URLs and confidence. Drop page text.
- **Customer chat:** keep account facts, promises made and the current issue. Drop earlier resolved issues to one line each.
- **Data pipeline agent:** keep schema, row counts and transformation decisions. Drop sample rows.
- **Long writing task:** keep the outline, style rules and what each section already says. Drop earlier drafts.

## Test it

Take a real long transcript, compact it, then ask the agent to continue. Check it:

- states the goal and constraints correctly,
- does not redo completed steps,
- uses exact identifiers,
- asks nothing the user already answered.

If any fail, fix the summary template, not the prompt.
