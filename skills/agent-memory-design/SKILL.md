---
name: agent-memory-design
description: Design how an LLM agent remembers things across turns and sessions. Use when adding memory to an agent, deciding what to store, where, and how to retrieve it, or when an agent forgets, repeats itself, or recalls stale facts. Covers working, episodic, semantic and procedural memory, write and recall rules, staleness, and privacy.
---

# Agent memory design

Memory is a product decision before it is a storage decision. Answer four questions first: what is worth remembering, who will read it, when is it read, and how does it get corrected.

## The four kinds

| Kind | Holds | Lives in | Example |
| --- | --- | --- | --- |
| Working | The current task | The context window | The file being edited, the plan so far |
| Episodic | What happened | A log, searched on demand | "Last Tuesday the deploy failed on the migration" |
| Semantic | Stable facts and preferences | A small index plus topic files | "User prefers tabs; staging URL is ..." |
| Procedural | How to do things | Skills, runbooks, checklists | "Release steps for this repo" |

Put each fact in one kind only. A fact stored in two places drifts apart.

## Write rules

1. Save what cannot be re-derived. Do not store code structure, git history or anything a tool call can fetch again.
2. One fact per entry, with a one-line description so it can be judged for relevance without opening it.
3. Store the why with the rule. "Do not mock the database, because mocks hid a broken migration" survives edge cases; the bare rule does not.
4. Convert relative dates ("next Thursday") to absolute dates when writing.
5. Check for an existing entry first. Update it rather than adding a near-duplicate.
6. Corrections beat confirmations. Record both when the user said "yes, exactly that", or the agent will only learn what not to do.

## Recall rules

- Keep an always-loaded index: one line per memory, no bodies. Load bodies on demand.
- Treat recalled memory as a claim about the past. Before acting on a named file, function or flag, check it still exists.
- When memory and the live state disagree, trust the live state and fix the memory.
- Cap recall. Three well-matched memories beat thirty loosely matched ones.

## Staleness and deletion

- Give every entry a type and a date. Review the oldest first.
- Delete memories that turn out to be wrong, without waiting for a cleanup pass.
- Let users see, edit and clear their memory. Never store secrets, tokens or other people's personal details.

## By use case

- **Coding agent:** semantic memory for conventions and user preferences; procedural memory for build and release steps; skip anything in the repo or its docs.
- **Support bot:** episodic memory per customer (past tickets, promises made), scoped by account so one customer never sees another's. Keep policy in procedural memory, not per-customer notes.
- **Research agent:** episodic memory of sources read and claims verified, with URLs, so it does not re-read or re-verify. Store confidence and date.
- **Personal assistant:** semantic memory of preferences, with an explicit confirm step before saving anything sensitive and an easy "forget that".
- **Multi-agent system:** a shared store for team facts, a private scratchpad per agent. Write through one owner to avoid conflicting edits.

## Retrieval choices

- Under a few hundred entries: an index file the model reads directly. No vector store needed.
- Large or fuzzy corpora: embeddings plus a keyword filter, then let the model pick from the top results.
- Always return the source and date with each recalled item.

## Failure checks

- Agent repeats a question the user already answered: nothing was written, or the index is not loaded.
- Agent acts on an outdated fact: no verify step, or no dates.
- Memory grows without limit: no dedupe and no deletion.
- Memory contradicts itself: one fact stored in two places.
