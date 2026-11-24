---
name: subagent-delegation
description: Decide when to split work across subagents and how to brief them so results come back usable. Use when designing a multi-agent system, when a task floods the main context with search or file reads, or when parallel independent work could save time. Also use to review delegation prompts that return vague or wrong results.
---

# Subagent delegation

A subagent is a fresh context that does a bounded job and returns a short result. Its main benefit is isolation: the noisy exploration stays in the subagent and only the answer reaches the parent. It costs extra tokens and starts with no knowledge of the conversation.

## Delegate when

- The work is a broad search where you only need the conclusion.
- Pieces are independent and can run in parallel.
- You want a second, uncorrelated look (review, verification).
- The output would otherwise flood the main context.

## Do not delegate when

- The task is small. A direct call is cheaper than a briefing.
- You are blocked on the answer right now and have nothing else to do.
- The pieces depend tightly on each other's intermediate state.
- Understanding the result requires context only the parent has, and you cannot write it down in a paragraph.

## Brief like a handoff to a stranger

The subagent has seen nothing. Include:

1. The goal and what it is for.
2. What is already known or ruled out, so it does not repeat it.
3. Exact scope: which paths, sources, time window.
4. The output format and a length cap ("under 200 words, file paths with line numbers").
5. Whether it may change things or only read.
6. How to report uncertainty: "say what you could not check".

Never write "based on your findings, fix it". Delegating understanding hands off the part that needed judgment. Do the synthesis yourself.

## Using results

- Treat the report as a claim, not a fact. For anything that matters, check the cited file or source.
- A subagent's summary describes what it intended, which may differ from what it did. Verify edits it claims to have made.
- Run independent subagents in the same step. Run dependent ones in sequence.

## Cost control

- Cap how many run at once and how deep they may nest.
- Give each a narrow tool set.
- Choose a smaller model for search and extraction, a larger one for review and synthesis.
- Stop early: tell each subagent when it has done enough.

## By use case

- **Codebase exploration:** one read-only subagent per question, each returning paths and a one-line finding.
- **Code review:** several reviewers with different focus (correctness, security, tests), then a verifier per finding.
- **Research:** one subagent per source set or sub-question, parent writes the synthesis.
- **Data processing:** shard the input, one worker per shard, parent merges.
- **Support triage:** a classifier subagent routes, a specialist answers, parent keeps the customer thread.

## Failure checks

- Results vague: the brief lacked scope or a format.
- Results duplicated: briefs overlapped; partition by path or topic.
- Parent context still bloated: subagents returned too much; add a length cap.
- Wrong conclusions confidently stated: no verification step on high-stakes claims.
