# CLAUDE.md

> **Verification Method**: Documentation sections use ONLY `mcp__simplewebfetch__simpleWebFetch` tool to navigate official documentation.
>
> **Sources**:
> - Claude Code: https://code.claude.com/docs
> - Anthropic Resources: official-claude-skills-guide.md
> - Platform Best Practices: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
>
> **Verification Legend**: `✓ VERIFIED` = Confirmed against official docs with source URL

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## The SAV Directory

**SAV = Self-Contained Archive and Verification**

This directory is the **knowledge preservation system** for the thecattoolkit_v3 project. It contains a complete, self-contained documentation system that can recreate the entire toolkit from scratch.

### Purpose

- Complete knowledge base for AI agent instruction projects
- Self-contained (no external dependencies except official Claude Code docs)
- Pattern library with proven implementations
- Can be copied to any new project and work standalone

**Key insight:** Over-constraining agents causes more failures than under-constraining. Modern models follow instructions precisely without micromanagement.

### Quick Reference Patterns

**Context Topology Primitives:**

| Primitive | Memory Mode | Best For |
|-----------|-------------|----------|
| Skill (Default) | Shared | Heuristics, code standards |
| Skill (frontmatter with context: fork) | Isolated | Specialists (Linter, Auditor) |
| Task (Subagent) | Forked | Heavy lifting (refactoring, tests) |
| Command | Injected | Entry points, workflows |

**Multi-Phase Delegation:** (⚠ CUSTOM pattern, uses official features)
- Subagent constraint: `Task → Task` forbidden (true subagent nesting)
- **⚠ CUSTOM** (Empirical finding): `Skill(context: fork)` spawns a subagent with an isolated context—does not inherit parent history. Chained forked skills are observed but should be treated as experimental and validated on target platforms.
- Use single Task with inline skills OR chained forked skills for multi-phase workflows
- Enables unbiased validation through isolated contexts

**Delta Standard:**
```
Good Component = Expert Knowledge - What Claude Already Knows
```

Keep: best practices, modern conventions, project-specific decisions, domain expertise
Remove: basic concepts, standard library docs, generic tutorials, Claude-obvious operations

The `sav/` directory is the core intellectual property - the accumulated knowledge from 33+ production skills and 2026 research findings.

---

## Content Attribution

Documentation uses inline badges to indicate source:
- `[OFFICIAL]` - Ground truth from Claude Code official documentation (code.claude.com)
- `[ANTHROPIC]` - Official Anthropic resources (resources.anthropic.com)
- `[COMMUNITY]` - Best practices from community/Platform docs
- `[FINDING]` - Personal research findings and empirical discoveries
