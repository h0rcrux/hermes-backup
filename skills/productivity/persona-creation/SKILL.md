---
name: persona-creation
description: Create a rich agent persona in SOUL.md by researching a real person's philosophy, quotes, and book influences
trigger: User asks to create a new agent personality or SOUL.md based on a specific thinker/investor/philosopher
---

# Persona Creation Workflow

## Process
1. Research the person's core philosophy, famous quotes, and recommended books
2. Structure into SOUL.md with: core principles, thinking framework, expression style, anti-patterns
3. Save supporting docs (investment philosophy, reading list) in `/opt/data/`
4. Update memory with persona note

## SOUL.md Structure
```
# Hermes Agent Persona — [Name/Nickname]
<!-- background context -->
## Core Identity (one sentence)
## Knowledge Roots (思想来源 tree)
## Principle 1-N (each with: quote block + bullet traits)
## Thinking Framework (判断框架)
## Expression Style (表达风格)
## Anti-Patterns (禁区)
## Quote Pool (语录池 by category)
```

## Key Elements
- **判断框架**: 3-5 questions the persona always asks before acting
- **语录池**: Categorized quotes (about investing, life, cognition, markets, reading)
- **Anti-patterns**: What this persona would NEVER do (e.g., no showing off, no anxiety-mongering)
- **Expression style**: Tone, vocabulary level, humor type, cultural references

## Example: Duan Yongping (大道无形我有型)
- 5 principles: 本分/平常心/能力圈/长期主义/做减法
- Knowledge roots: Graham → Fisher → Buffett → Munger + Collins + Welch + Cialdini
- Location: `/opt/data/SOUL.md` + `duan-yongping-investment-philosophy.md` + `duan-yongping-reading-list.md`
