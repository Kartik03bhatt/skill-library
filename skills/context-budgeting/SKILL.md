---
name: context-budgeting
description: Decide what goes into an LLM's context window and how much space each part gets. Use when an agent's prompt is bloated, quality drops on long inputs, tool outputs are huge, or you are designing retrieval, tool results and instructions for a new agent.
---

# Context budgeting

The context window is working memory with a cost per token and a quality curve. Treat it like a budget: every token should earn its place. More context is not better context; irrelevant text dilutes attention and raises cost.

## Budget by section

Give each section a rough share and a hard cap:

| Section | Share | Rule |
| --- | --- | --- |
| Instructions and tool definitions | 5 to 15% | Stable, cached. Cut anything the model already does by default. |
| Standing knowledge (memory, docs) | 10 to 20% | Index first, bodies on demand. |
| Task state (plan, notes) | 5 to 10% | Rewritten, not appended. |
| Conversation and tool results | the rest | Bounded per call; cleared when stale. |
| Reserve for output and reasoning | 10 to 20% | Never spend it on input. |

## Tool results are the usual culprit

- Cap output per call and say so ("showing first 200 of 3,412 lines; narrow the query").
- Return what was asked. Filter, paginate and summarize in the tool, not in the model.
- Prefer identifiers plus a fetch tool over inlining everything. Return file paths and let the agent open the one it needs.
- Strip markup, boilerplate and repeated headers before returning.

## Retrieval

- Retrieve few, relevant chunks. Start with 3 to 5 and measure before raising.
- Put the most relevant material at the start or end of the context, not buried in the middle.
- Include source and date with each chunk so the model can weigh them.

## Instructions

- State rules once. Repeated rules drift into contradictions.
- Explain the reason behind a rule in a sentence; it generalizes better than a list of cases.
- Move rarely used procedures into skills loaded on demand rather than the system prompt.

## Measure

- Count tokens per section on real runs, not guesses.
- Plot task success against context size. If success falls as context grows, you have a dilution problem, not a limit problem.
- Track cost per completed task, not per call.

## By use case

- **Coding agent:** project guide in the system prompt (short), file contents on demand, test output truncated to failures.
- **RAG assistant:** small top-k, rerank, drop chunks below a relevance threshold rather than filling to the limit.
- **Voice or realtime agent:** latency is the budget. Keep the prompt short and the cached prefix stable.
- **Multi-step workflow:** pass forward a structured state object, not the full prior transcript.
- **Analysis over large documents:** map-reduce. Summarize chunks separately, then reason over the summaries.

## When over budget

1. Clear stale tool output.
2. Replace pasted content with references.
3. Split the task across subagents (see `subagent-delegation`).
4. Compact (see `context-compaction`).
