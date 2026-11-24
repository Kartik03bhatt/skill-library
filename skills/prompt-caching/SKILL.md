---
name: prompt-caching
description: Structure LLM prompts so the provider's prompt cache hits, cutting cost and latency. Use when building or debugging an agent or chat app that resends a long prefix every turn, when cache hit rates are low, or when deciding where to put cache breakpoints. Written around the Anthropic API; the ideas apply to other providers.
---

# Prompt caching

A prompt cache reuses the work done on an identical prompt prefix. Cached input is billed far below normal input and returns faster. The only thing that matters is whether the start of the request is byte-identical to a recent request.

Check current numbers in the provider's docs before quoting them. They change.

## Core rule

Put what is stable first and what changes last.

Order for the Anthropic API: tools, then system prompt, then messages. A change at any point invalidates everything after it, so a changed tool definition invalidates the system prompt and the whole conversation cache.

## Layout

1. Tool definitions: sorted and serialized deterministically.
2. System prompt: static instructions, style rules, reference material.
3. Long stable context: documents, a codebase summary, few-shot examples.
4. Conversation history: append only.
5. The new user turn: always uncached, always last.

## Breakpoints (Anthropic)

- Mark a block with `cache_control: {"type": "ephemeral"}`. Everything up to and including that block is cached.
- There is a small limit on breakpoints per request (four at the time of writing).
- Place one at the end of the static prefix. In a multi-turn loop, place one on the latest message so the next turn reads the whole history from cache.
- A prefix below the model's minimum cacheable length is silently not cached. Check the minimum for your model.
- Default lifetime is about five minutes, refreshed on each hit. A longer option exists at a higher write price. Choose it only when requests are spaced further apart than the short lifetime.

## What silently breaks the cache

- A timestamp, request ID or random value near the top of the system prompt.
- Reordering tools or JSON keys between requests.
- Editing or deleting earlier messages in the history (this includes tool-result trimming; see `context-compaction`).
- Switching model, changing thinking settings or toggling features that alter the prefix.
- Per-user text placed before shared text.

Move anything volatile to the end, or into the latest user message.

## Verify, do not assume

Read the usage fields on every response: tokens written to cache, tokens read from cache, and uncached input. Log the ratio. A healthy agent loop shows large reads and small uncached input after turn one. If reads are zero, diff two consecutive requests byte for byte.

## Cost reasoning

Writing costs a little more than normal input; reading costs a small fraction of it. Caching pays off once a prefix is reused a couple of times inside its lifetime. It loses money on prefixes used once.

## By use case

- **Coding agent:** tools and system prompt cached; the repo summary cached; each turn appends tool results. Keep a breakpoint on the last message.
- **Document Q&A:** put the document first, the question last, and cache the document across many questions.
- **Support bot:** shared policy text first, customer-specific data after the breakpoint.
- **Batch jobs:** one shared instruction prefix, many different items. Send the first request, wait for the write to complete, then fan out so the rest read it.
- **Low-traffic app:** if requests are minutes apart, the cache expires between them. Consider the longer lifetime or accept no caching.

## Checklist

- [ ] Volatile values removed from the prefix
- [ ] Tools serialized in a fixed order
- [ ] Breakpoint at end of static prefix and on the latest message
- [ ] Prefix longer than the model's minimum
- [ ] Cache read and write tokens logged per request
