# skill-library

Agent Skills for building LLM agents that remember, cache, compact and recover. Each skill is a folder with a `SKILL.md` (frontmatter plus instructions) that works with Claude Code and other tools that read the Agent Skills format.

## Skills

| Skill | Use it to |
| --- | --- |
| [`agent-memory-design`](skills/agent-memory-design/SKILL.md) | Decide what an agent remembers, where, and how it is recalled and corrected |
| [`prompt-caching`](skills/prompt-caching/SKILL.md) | Lay out prompts so the provider cache hits, and verify it does |
| [`context-compaction`](skills/context-compaction/SKILL.md) | Shrink a long context without losing the goal, constraints or state |
| [`context-budgeting`](skills/context-budgeting/SKILL.md) | Allocate the context window across instructions, knowledge and tool output |
| [`subagent-delegation`](skills/subagent-delegation/SKILL.md) | Split work across subagents and brief them well |
| [`agent-checkpointing`](skills/agent-checkpointing/SKILL.md) | Make long runs resumable, idempotent and safe to retry |

Every skill ends with a "By use case" section: coding agents, support bots, research agents, data pipelines and others.

## Install

Copy a skill folder into your project or user skills directory:

```sh
cp -r skills/prompt-caching ~/.claude/skills/        # all projects
cp -r skills/prompt-caching .claude/skills/          # one project
```

## Add a skill

Copy `template/SKILL.md` into `skills/<name>/`, fill it in, and add a row to the table above. The folder name must match `name` in the frontmatter, and `description` should say both what the skill does and when to use it.

## Notes

Provider details (limits, prices, feature names) change. The skills point to the idea and say when to check current docs rather than hard-coding numbers.

## Licence

MIT. See `LICENSE`.
