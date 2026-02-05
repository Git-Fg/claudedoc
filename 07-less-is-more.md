---
number: 07
title: Less is More
audience: all
related: [00, 01, 02, 04]
last-reviewed: 2026-02-05
---

# 07 - Less is More: A Comprehensive Guide to AI Agent Minimalism

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [The Research Foundation](#the-research-foundation)
   - [Anthropic: Building Effective Agents](#anthropic-building-effective-agents)
   - [Context Poisoning: The Chroma Study](#context-poisoning-the-chroma-study)
   - [Tool Overload: Performance Impact](#tool-overload-performance-impact)
   - [Community Consensus](#community-consensus)
   - [Prompt Engineering Research](#prompt-engineering-research)
   - [Theoretical Foundations 2026](#theoretical-foundations-2026)
     - [Agent Complexity Law](#agent-complexity-law)
     - [Agency Efficiency Principle (LIMI)](#agency-efficiency-principle-limi)
     - [Chain of Draft (CoD)](#chain-of-draft-cod)
3. [Core Principles](#core-principles)
   - [The Nécessaire et Suffisant Framework](#the-nécessaire-et-suffisant-framework)
   - [Context Budget Thinking](#context-budget-thinking)
   - [The 2026 Paradigm Shift](#the-2026-paradigm-shift)
   - [Context Pruning & Goldilocks Zone](#context-pruning--goldilocks-zone)
   - [Just-in-Time Context](#just-in-time-context)
4. [When Simplicity Wins vs Structure Needed](#when-simplicity-wins-vs-structure-needed)
5. [Toolkit Focus Strategy](#toolkit-focus-strategy)
   - [The 10-Tool Maximum Rule](#the-10-tool-maximum-rule)
   - [MCP Server Best Practices](#mcp-server-best-practices)
6. [Prompt Minimalism Patterns](#prompt-minimalism-patterns)
   - [Pattern 1: Direct Intent](#pattern-1-direct-intent)
   - [Pattern 2: Intent + Context + Constraints](#pattern-2-intent--context--constraints)
   - [Pattern 3: Objective + Success Criteria](#pattern-3-objective--success-criteria)
   - [Pattern 4: Natural Language Delegation](#pattern-4-natural-language-delegation)
   - [Pattern 5: Chain of Draft](#pattern-5-chain-of-draft)
   - [Pattern 6: Canonical Examples](#pattern-6-canonical-examples)
7. [Component Minimalism Guidelines](#component-minimalism-guidelines)
   - [Skills: Concise and Targeted](#skills-concise-and-targeted)
   - [Commands: Single Responsibility](#commands-single-responsibility)
   - [Agents: Flat Architecture](#agents-flat-architecture)
8. [The Iterative Workflow](#the-iterative-workflow)
9. [Measurement Over Dogma](#measurement-over-dogma)
10. [Codebase Optimization for Agents](#codebase-optimization-for-agents)
11. [Claude Code & OpenCode Implementation](#claude-code--opencode-implementation)
   - [Double Mode Strategy](#double-mode-strategy)
12. [Red Flags: Over-Engineering Checklist](#red-flags-over-engineering-checklist)
13. [Complete Examples](#complete-examples)
14. [References](#references)

---

## Executive Summary

The optimization of AI agents is not a race toward complexity but a pursuit of **elegant simplicity**. Research from Anthropic, empirical studies from the AI community, and production benchmarks consistently demonstrate that simpler systems outperform complex ones across key metrics: accuracy, cost, latency, and maintainability.

**Key Insights:**

- Model precision drops from 78% to 13% when tool count exceeds 100[^1]
- Context window bloat reduces accuracy by ~30% when using full history vs targeted context[^2]
- Simple prompts often outperform complex structured ones for modern models (Claude 4+, GPT-4.1+)[^3]
- Over-constraining agents causes more failures than under-constraining[^4]

This document provides a comprehensive methodology for creating optimal, minimal prompts and components, explaining **why** simplicity works and **how** to implement it effectively.

---

## The Research Foundation

### Anthropic: Building Effective Agents

According to Anthropic's research report "Building Effective Agents," the most successful agent implementations rely not on complex frameworks but on **simple, composable patterns**[^5]. Their study with dozens of industrial teams revealed:

| Finding | Implication |
|:--------|:------------|
| **Simplicity beats complexity** | Complex abstractions harm performance |
| **Minimal systems outperform** | Simpler architectures are more reliable |
| **Trust the model's intelligence** | Modern models follow instructions precisely |

**Key Quote:**
> "The most effective implementations were not the most complex, but the simplest that could solve the problem."

This research validates a fundamental shift: from micromanaging agents with rigid constraints to providing clear objectives and trusting their reasoning capabilities.

### Context Poisoning: The Chroma Study

The Chroma research team conducted a comprehensive study on what they term "Context Rot"—the degradation of model performance as context length increases[^6]. Their findings are alarming:

**The Numbers:**
- Adding full chat history (~113,000 tokens) degrades accuracy by **~30%** compared to targeted context (~300 tokens)
- All major models tested (GPT-4.1, Claude 4, etc.) suffer from context rot
- The assumption that "more context = better results" is a myth

**Mechanisms of Context Poisoning:**

1. **Context Window Bloat**
   - Every tool, detailed instruction, or complex structure consumes precious tokens
   - More tokens = less space for actual reasoning
   - Token usage per request can explode from 1,084 to 127,315 with excessive tooling[^7]

2. **Decision Fatigue**
   - Each additional choice increases cognitive load
   - More options = longer execution times
   - Inconsistent choices due to decision overload

3. **Interference Effects**
   - Models overestimate the importance of old information
   - Too many tools induce unnecessary tool usage
   - Information from different sources creates conflicts

**Practical Implication:**
> "Less is better because the less tokens you put out, the less context window you're eating up and the better chances you have for the model to reach task completion."[^8]

### Tool Overload: Performance Impact

Research from Joshua Berkowitz on solving tool overload reveals catastrophic performance degradation[^9]:

| Tool Count | Precision | Notes |
|:-----------|:----------|:------|
| 10 tools | 78% | Baseline performance |
| 50 tools | ~45% | Significant decline |
| 100+ tools | 13% | Catastrophic failure |

**Additional Findings:**
- State-of-the-art retrieval models achieve <35% precision in tool selection
- Poor retrieval quality directly reduces task success rates by 10-20%
- Latency increases with each tool considered before selection

**The 10-Tool Threshold:**
Beyond 10 tools, precision begins to decline significantly[^10]. This creates a practical upper bound for static tool loading.

### Community Consensus

A Reddit discussion on over-engineering agents (92 upvotes) captures the community sentiment[^11]:

> "From our own experience, over-engineering is poison for AI agents. Greater stability arises from minimizing the number of components rather than complicating them. Complexity often undermines consistent behavior."

Another developer summarizes their approach:
> "We concentrated on maximum autonomy with minimal constraints, moving away from ReAct. The end product operates on simple text commands, with core code comprising roughly 2,000 lines."

This represents a broad consensus: the community is moving away from complex agent architectures toward simpler, more autonomous systems.

### Prompt Engineering Research

A synthesis of 1,500+ academic papers on prompt engineering concludes that **short, well-structured, dense prompts outperform verbose ones** for equal or better quality at lower cost[^12].

**Key Findings:**
- Zero-shot simple prompts can outperform few-shot complex prompts in some scenarios
- For modern powerful models, simple prompts sometimes beat complex styles in cost/complexity tradeoffs
- Concise prompts with contextual relevance beat lengthy context-heavy prompts[^13]

### Theoretical Foundations 2026

**[FINDING] The Subtractive Turn in AI Architecture**

The AI development ecosystem is undergoing a critical structural transformation. During 2023-2024, the industry was dominated by an additive philosophy: to improve agent reliability, the standard response was to add layers of abstraction. Developers stacked complex prompt chains, rigid sub-agent hierarchies, voluminous skill registries, and heavy orchestration frameworks. The underlying hypothesis postulated that denser infrastructure equaled superior intelligence.

However, research converging in late 2025 and early 2026 indicates this approach reached a point of diminishing—if not negative—returns. We are witnessing what can be termed the **"subtractive turn"** in AI system architecture.

#### Agent Complexity Law

A major contribution to recent literature is the formalization of the **Agent Complexity Law**. Proposed in research published mid-2025, this law establishes an inverse relationship between model capability and architectural scaffolding necessity[^24].

**The Law States:**
> "The performance gap between agents of varying complexity—from simple to sophisticated designs—will narrow as the core model improves, eventually converging toward a negligible difference."

**Implications:**
- Investment in learning and implementing complex structures (sub-agents, explicit feedback loops) experiences rapid depreciation
- As you migrate to Claude 3.7, GPT-5, or newer models, complex scaffolding becomes technical debt
- The Lita framework validated that "thin agents" (minimalist agents with basic capabilities) reveal the full coding competence of modern LLMs[^24]

**Research Evidence:**
Researchers demonstrated through the Lita ("Light Agent") framework that a minimalist design—a thin agent with basic capabilities—was sufficient to reveal the full agentic coding capabilities of modern LLMs. Heavy frameworks attempting to "micromanage" the AI's thought process added no significant value and sometimes introduced noise that degraded performance.

| Architecture | Performance | Token Cost | Latency |
|:-------------|:------------|:-----------|:--------|
| Heavy Framework (200+ lines config) | Baseline | 127k tokens | High |
| Thin Agent (minimal config) | Equal or better | 3k tokens | Low |
| **Improvement** | **0-5%** | **-97.6%** | **-75%** |

#### Agency Efficiency Principle (LIMI)

The **LIMI Study** (Less Is More for Intelligent Agency) challenges the dogma that "more data" or "more examples" always equals better performance[^25].

**Striking Results:**

| Training Approach | Samples | AgencyBench Performance |
|:------------------|:--------|:------------------------|
| Traditional (generic) | 10,000 | 47.8% |
| **LIMI (curated)** | **78** | **73.5%** |
| **Improvement** | **-99.2%** | **+53.7%** |

**Key Insights:**
1. **Quality over Quantity**: Machine autonomy emerges not from data volume, but from strategic curation of high-quality agentic demonstrations
2. **The 78-Sample Threshold**: Using only 78 carefully designed training samples, the LIMI model achieved 73.5% on AgencyBench evaluation
3. **Canonical Examples Pattern**: One perfect example embodies more value than 1000 words of abstract instructions

**Practical Application:**
A CLAUDE.md file containing three canonical examples of your coding style is infinitely more powerful than 50 pages of exhaustive documentation. LLMs are pattern-matching machines—they excel at imitating implicit style in an example rather than applying explicit abstract rules.

#### Chain of Draft (CoD)

**The Paradox:** To obtain more sophisticated behaviors, we must give less directive instructions about the "how" and focus only on the "what."

**Mechanism:**
Chain of Draft forces the AI to adopt rapid cognition behavior. When an expert solves a problem, they don't articulate every logical step with complete sentences. They use mental shorthand: keywords, sketches, partial equations. CoD forces this behavior.

**Comparison:**

| Aspect | Chain of Thought (Traditional) | Chain of Draft (Minimalist) |
|:-------|:-------------------------------|:----------------------------|
| **Instruction** | "Think step by step. Explain your reasoning." | "Use CoD. Dense reasoning, <5 words per step." |
| **Output Example** | "I must first check authentication. The auth.ts file seems to use JWT..." | "Check auth.ts → JWT found → Expiry check missing." |
| **Token Volume** | High (~200 tokens/response) | Very Low (~40 tokens/response) |
| **Latency** | Slow | Fast (50-75% reduction) |
| **Cognitive Load** | Risk of "verbosity" that dilutes meaning | Focus on semantic density |
| **Accuracy** | Baseline | Maintained or improved[^26] |

**Research Results:**
On reasoning tasks (GSM8k arithmetic and common sense), CoD maintained or improved accuracy (91.4% on GSM8k) while reducing token usage by over 80%[^26].

**Implementation:**
Add this instruction to any complex task:
```
"Use Chain of Draft: dense reasoning, maximum 5 words per step."
```

This forces the model to concentrate computational power on pure logic rather than syntactic generation—the essence of "Less is More" applied to latency.

---

## Core Principles

### The Nécessaire et Suffisant Framework

This framework, derived from mathematical logic, states that a solution should be **necessary** (required) and **sufficient** (enough)—no more, no less.

**Comparison:**

```markdown
❌ Complex Approach (Avoid)
- 15+ tools loaded simultaneously
- Prompts structured of 50+ lines
- Multiple agent hierarchies
- Overloaded MCP servers
- Preemptive handling of all edge cases

✅ "Less is More" Approach
- 3-5 essential tools per task
- Prompts concise of 1-3 lines when possible
- Flat architecture when sufficient
- Dynamic tool loading only when needed
- Reactive constraints added only after observing failures
```

### Context Budget Thinking

Treat the context window as a **finite budget** rather than an infinite resource:

**The Math:**
- Context window = Limited tokens (e.g., 200k for Claude 3 Opus)
- Every token spent on instructions = Token not spent on reasoning
- Every tool loaded = Cognitive overhead for selection

**Budget Allocation Priority:**
1. Task objective (highest priority)
2. Critical constraints (must-have)
3. Relevant context (supporting info)
4. Tool definitions (only essential)
5. Nice-to-have instructions (lowest priority—often omitted)

**Key Insight:**
> "The art of context engineering is finding the smallest set of high-signal tokens possible to achieve the desired outcome."[^14]

### The 2026 Paradigm Shift

| Aspect | Old Way (2024) | New Way (2026) |
|:-------|:---------------|:---------------|
| **Agent Management** | Carefully constrain | Trust by default |
| **Instructions** | List forbidden actions | State objectives and alternatives |
| **Workflows** | Rigid step-by-step scripts | Autonomous decision-making |
| **Communication** | Magic syntax and formatting | Natural language |
| **Supervision** | Micromanage every operation | Reactive constraints only |

**Why This Matters:**

Modern models (Claude 4+, GPT-4.1+) are instruction-following machines. They read what you write and execute faithfully. However, **over-constraint causes rigidity failure**—when you script every step, the agent cannot adapt when reality diverges from the script.

**Concrete Example:**

❌ **2024-style failure:** You write "Step 1: Read file A. Step 2: Edit line 5." But file A was deleted. The agent tries to read the deleted file, fails, and halts.

✅ **2026-style success:** You write "Update the auth module to use the new API. Handle any missing files by checking git status first." The agent notices the file is gone, checks git status, and intelligently adjusts.

### Context Pruning & Goldilocks Zone

**[FINDING] Attention Budget Management**

Transformers have a finite context window, but more importantly, a **finite attention budget**. As the context window fills, the model's ability to retrieve specific information degrades (the "needle in a haystack" problem), or the model may begin to confuse contradictory instructions buried in the history.

**Context Pruning Discipline:**

The "unlearning" process involves stopping the view of the context window as unlimited storage. Adopt a discipline of **context pruning**[^27].

**The Pruning Principle:**
Every token that is not a "strong signal" is noise. Outdated technical documentation, verbose error logs, or irrelevant past conversations dilute agent performance.

**The Goldilocks Zone:**

The goal is to find the "just right" amount of context:

| Zone | Context Amount | Result |
|:-----|:---------------|:-------|
| **Too Little** | <5% of capacity | Hallucinations due to insufficient information |
| **Goldilocks** | 20-50% of capacity | Optimal signal-to-noise ratio |
| **Too Much** | >70% of capacity | Confusion and degraded performance |

**Practical Guidelines:**

1. **Remove obsolete context:** Delete outdated documentation, irrelevant conversations, superseded requirements
2. **Summarize long histories:** Use `/reset` or start fresh sessions frequently
3. **Explicit context selection:** Use `@file` or manual file selection over automatic RAG
4. **One context per concern:** Don't mix unrelated topics in the same session

**Signs You Need Pruning:**
- Agent starts asking questions you already answered
- Responses become generic or off-topic
- Agent contradicts earlier constraints
- Latency increases significantly

### Just-in-Time Context

**[FINDING] Dynamic vs Static Context Loading**

A "thick" architecture loads all context at startup. A "thin" architecture loads context on demand (Just-in-Time).

**Comparison:**

| Approach | Example | Result |
|:---------|:--------|:-------|
| **"Thick Agent"** | "Here's the entire /src folder, analyze and find the bug" | Context polluted with irrelevant files |
| **"Thin Agent"** | "Here are grep and cat tools. Explore to find the bug" | Only relevant context enters working memory |

**The Just-in-Time Strategy:**

```markdown
❌ Don't: "Read all these files [20 files attached]"

✅ Do: "Find the bug in the authentication system"
   → Agent uses tools to explore
   → Agent reads only auth.ts, user.ts, jwt.ts
   → Context stays focused on relevant files
```

**Why This Works:**

1. **Context Hygiene:** Forces the agent to actively select what it needs
2. **Reduced Hallucinations:** Less noise = fewer false pattern matches
3. **Lower Token Costs:** Pay only for context actually used
4. **Better Reasoning:** Working memory stays uncluttered

**Implementation:**

Instead of pre-loading context, provide tools and let the agent explore:

```markdown
Available tools:
- Read(path) - Read specific files
- Grep(pattern) - Search codebase
- Bash(cmd) - Execute shell commands

Task: Debug why login fails for users with 2FA enabled
```

The agent will:
1. Grep for "2FA" and "login" to find relevant files
2. Read only auth.ts and user.ts
3. Identify the bug in the 2FA verification flow
4. Propose a fix

This maintains a clean, focused context throughout the interaction.

---

## When Simplicity Wins vs Structure Needed

### Simplicity Wins

Use minimal prompts and simple structures when:

| Scenario | Why Simplicity Works |
|:---------|:---------------------|
| **Discovery/Exploration** | Natural language adapts organically to revelations |
| **Requirements Gathering** | AskUserQuestion handles interaction better than scripted branches |
| **Code Review/Analysis** | High-freedom tasks benefit from model's reasoning |
| **One-off Coding Tasks** | Context-dependent decisions need flexibility |
| **Brainstorming/Ideation** | Creative tasks need room for exploration |
| **Bug Investigation** | Unknown problems require adaptive approaches |

### Structure Needed

Add complexity only when:

| Scenario | Why Structure Helps |
|:---------|:--------------------|
| **Repeatable Workflows** | Predefined outputs require consistency |
| **System Integrations** | CRMs, support platforms need reliability |
| **Production-Critical Code** | Consistency is essential for stability |
| **Explicit Chain-of-Thought** | Some reasoning must be visible/auditable |
| **Safety-Critical Systems** | Deviation is dangerous |
| **Regulatory Compliance** | Must follow exact procedures |

**Decision Framework:**

```
Is the output format strictly predefined? 
  → Yes → Use structured prompts
  → No → Is the task safety-critical?
      → Yes → Use structured prompts
      → No → Start with simple prompt
```

---

## Toolkit Focus Strategy

### The 10-Tool Maximum Rule

Research shows that beyond 10 tools, precision declines significantly[^15]. This creates practical constraints:

**Recommended Hierarchy:**

| Tier | Tool Count | Purpose | Loading Strategy |
|:-----|:-----------|:--------|:-----------------|
| **Essential** | 3-5 tools | Cover 80% of use cases | Statically loaded |
| **Specialized** | 2-3 tools | Domain-specific needs | Loaded per workflow |
| **Contextual** | Variable | Task-specific requirements | Dynamically selected |

**Implementation:**

```typescript
// ❌ Bad: Static loading of 20+ tools
const tools = [tool1, tool2, ..., tool20];

// ✅ Good: Dynamic loading based on task
const relevantTools = await selectToolsForTask(userQuery); // 3-5 tools max
```

### MCP Server Best Practices

For MCP (Model Context Protocol) server efficiency[^16]:

**Design Principles:**

1. **Fewer tools + richer parameters**
   - Prefer polymorphic tools with many parameters
   - Avoid exposing every API function as a separate tool

2. **Tools as "agent stories"**
   - Treat tools as packaged narratives, not 1:1 API wrappers
   - A tool should describe what it enables, not just what it calls

3. **Trim response payloads**
   - Reduce JSON verbosity
   - Remove redundant metadata
   - Return only what the agent needs

4. **Actionable errors**
   - Errors should suggest next steps
   - Help models self-correct

5. **Stable tool lists**
   - Don't change available tools mid-session
   - Preserves caching and consistency

**Key Quote:**
> "Less is better because the less tokens you put out, the less context window you're eating up and the better chances you have for the model to reach task completion."[^17]

---

## Prompt Minimalism Patterns

### Pattern 1: Direct Intent

**Use for:** Simple, well-understood tasks

```javascript
// ❌ Over-specified and constraining
"Écris une fonction Python qui calcule la moyenne d'une liste de nombres, 
en utilisant une approche fonctionnelle avec des lambda, 
en gérant les cas d'erreur, avec une documentation complète, 
en suivant PEP8, et en retournant None si la liste est vide"

// ✅ Simple and effective
"Calcule la moyenne d'une liste de nombres en Python"
```

**Why it works:** Modern models understand intent and apply best practices automatically. Over-specification limits their ability to choose optimal approaches.

### Pattern 2: Intent + Context + Constraints

**Use for:** Tasks requiring background knowledge and boundaries

```markdown
Goal: Ajoute l'endpoint POST /users avec validation email/password
Context: API REST FastAPI avec SQLAlchemy, repo dans ./app
Constraints: Ne touche pas aux migrations, utilise les modèles existants
```

**Structure:**
- **Goal** (1 line): What you want
- **Context** (1 line): Relevant background
- **Constraints** (1-2 lines): Key limitations

### Pattern 3: Objective + Success Criteria

**Use for:** Complex tasks needing clear completion definition

```markdown
Objective: Implement user authentication with JWT tokens
Success Criteria:
- Returns token on valid credentials (200)
- Returns 401 on invalid credentials
- Follows existing error handling patterns in auth/
- All existing tests pass
```

**Structure:**
- **Objective**: High-level goal
- **Success Criteria**: Measurable outcomes defining "done"

**Alternative Format (Mission Control Pattern):**

```markdown
<mission_control>
<objective>Implement JWT authentication endpoint</objective>
<success_criteria>
- Returns token on valid credentials
- Returns 401 on invalid credentials
- Follows existing auth patterns
- All tests pass
</success_criteria>
</mission_control>
```

### Pattern 4: Natural Language Delegation

**Use for:** Tasks requiring autonomy and adaptation

```markdown
"Refactor this authentication module to use the new security patterns. 
Handle any breaking changes by checking the migration guide first. 
Ensure all existing tests still pass."
```

**Characteristics:**
- Describes outcome, not procedure
- Mentions resources to consult
- States success criteria implicitly or explicitly
- Trusts model to determine steps

### Pattern 5: Chain of Draft

**Use for:** Complex reasoning tasks requiring speed and efficiency

The Chain of Draft (CoD) pattern forces dense, abbreviated reasoning similar to expert human cognition. Instead of verbose step-by-step explanations, the model uses terse notation.

**The Instruction:**
```markdown
"Use Chain of Draft: dense reasoning, maximum 5 words per step."
```

**Comparison:**

| Aspect | Chain of Thought (Traditional) | Chain of Draft (Minimalist) |
|:-------|:-------------------------------|:----------------------------|
| **Instruction** | "Think step by step. Explain your reasoning." | "Use CoD. Dense reasoning, <5 words per step." |
| **Output Example** | "First, I need to check the authentication module. The auth.ts file appears to use JWT tokens for session management. I should verify the token expiry logic..." | "Check auth.ts → JWT found → Expiry check missing." |
| **Token Volume** | ~200 tokens/response | ~40 tokens/response |
| **Latency** | Slow | Fast (50-75% reduction) |
| **Semantic Density** | Low (verbose explanations) | High (concentrated meaning) |

**Research Results:**
On reasoning benchmarks (GSM8k arithmetic and common sense), CoD maintained or improved accuracy (91.4% on GSM8k) while reducing token usage by over 80%[^26].

**Example Applications:**

```markdown
**Debugging:**
"Use CoD: Trace this error → Check auth flow → Identify null pointer"

**Refactoring:**
"Use CoD: Analyze coupling → Extract interface → Move to module"

**Architecture:**
"Use CoD: Review patterns → Identify bottleneck → Propose solution"
```

**Why It Works:**
- Forces model to concentrate on logic, not syntax
- Reduces cognitive "verbosity" that dilutes meaning
- Faster iteration cycles
- Lower API costs

### Pattern 6: Canonical Examples

**Use for:** Establishing patterns, styles, and conventions

**The Canonical Example Principle:**
One perfect code example is more valuable than 1000 lines of abstract rules. LLMs are pattern-matching machines—they excel at imitating implicit style in an example.

**Structure:**

```markdown
## Canonical Example

See @src/utils/result.ts for the definitive pattern.

Key characteristics to emulate:
- Error handling with Result<T, E> type
- Exhaustive pattern matching
- Early returns for error cases
- Descriptive error messages
```

**Comparison:**

| Approach | Content | Effectiveness |
|:---------|:--------|:--------------|
| **Abstract Rules** | "Use functional programming. Prefer immutability. Handle errors explicitly." | Low (vague, open to interpretation) |
| **Canonical Example** | Actual code file demonstrating all principles | High (concrete, imitable) |

**Implementation:**

1. **Create a canonical example file:**
```typescript
// src/utils/canonical-example.ts
// This file demonstrates our coding standards

export function processUserData(data: unknown): Result<User, ValidationError> {
  // Early validation
  if (!data || typeof data !== 'object') {
    return Result.err(new ValidationError('Invalid data structure'));
  }
  
  // Exhaustive checks
  const { name, email, age } = data as Record<string, unknown>;
  if (!name || typeof name !== 'string') {
    return Result.err(new ValidationError('Name is required'));
  }
  
  // Success path
  return Result.ok({ name, email, age });
}
```

2. **Reference in CLAUDE.md:**
```markdown
## Coding Standards

Follow the pattern demonstrated in @src/utils/canonical-example.ts:
- Use Result<T, E> for error handling
- Validate early, return early
- Be exhaustive in type checking
- Use descriptive error messages
```

**Benefits:**
- Concrete reference that doesn't degrade with context compaction
- Self-documenting through working code
- Easier to update than abstract rules
- Model can infer implicit patterns

**Research Evidence:**
The LIMI study demonstrated that 78 carefully curated examples outperformed 10,000 generic training samples by 53.7%[^25]. Quality and curation matter more than volume.

---

## Component Minimalism Guidelines

### Skills: Concise and Targeted

**Target Specifications:**

| Aspect | Simple Skill | Complex Skill |
|:-------|:-------------|:--------------|
| **Length** | 20-50 lines | 100+ lines |
| **Structure** | Natural language guidelines | Step-by-step enforcement |
| **Best For** | Discovery, heuristics, patterns | Validation, strict workflows |
| **Tools** | 3-5 essential | Dynamically selected |

**The "What I Do / When to Use Me" Pattern:**

```yaml
---
name: fastapi-crud-conventions
description: Follow our FastAPI CRUD patterns
---

## What I do
- Use `router.prefix` for resource routes
- Expose only `/items/{id}` and `/items/` endpoints unless explicit requirement
- Always validate with Pydantic V2 models
- Return 404 if not found, 422 for validation errors, 400 for bad requests

## When to use me
Use for any new CRUD route or DTO.
Ask if pagination/search is required before implementing.
```

**Why this works:**
- Ultra-concise (~20 lines)
- Clear value proposition
- Precise trigger conditions
- Defers to agent for judgment on edge cases

**Another Example (Refactoring Skill):**

```yaml
---
name: refactor-typescript-strict
description: Refactor existing TS files toward strict mode safety
---

## What I do
- Enable `strict: true` if not already present, minimally
- Add explicit return types where `noImplicitAny` would break
- Replace `any` by `unknown` + guards when possible
- Update tests to cover new type errors

## When to use me
Use when you have a working TS module and want to make it safer without changing behavior.
Ask if breaking changes are allowed before touching exported APIs.
```

### Commands: Single Responsibility

**Principles:**
- One command = one purpose
- Avoid branching logic unless necessary
- Let AskUserQuestion handle interaction complexity

**Example: The 22-Line Interview Command**[^18]

```yaml
---
argument-hint: [instructions]
description: Interview user in-depth to create a detailed spec. Use when requirements are unclear. Not for tasks with rigid output formats.
allowed-tools: AskUserQuestion, Write, Read, Glob
---

Instructions: $ARGUMENTS

Follow user's instructions and use AskUserQuestionTool to interview him in depth so you can later write a complete specification document and save it to a file.

When you ask questions with AskUserQuestionTool:
- Always ask exactly one question at a time
- Offer clear, meaningful answer choices plus "Something else" option
- Design options to help orient the user, not constrain them
- Never ask questions you could answer yourself by using fetch or reading files
- Include meaningful examples when propositions are too abstract

Continue this one-question-at-a-time process until you have enough detail to write a thorough specification.
```

**What 22 lines accomplish:**
- No predetermined question sequences—adapts organically
- Built-in AskUserQuestion tool handles interaction complexity
- Accomplishes what 200-line scripted commands attempt

**Why Simple Commands Win:**

| Aspect | Simple Command | Complex Command |
|:-------|:---------------|:----------------|
| **Adaptability** | Pivots based on user revelations | Forces predetermined path |
| **Maintenance** | 10-50 lines = easy updates | 200+ lines = hard to maintain |
| **Effectiveness** | AskUserQuestion provides structure | Hardcoded branches miss edge cases |
| **Focus** | Guides goal, not process | Micromanages steps |

### Agents: Flat Architecture

**Recommended Structure:**

```
Main Agent (Dev Lead)
├── Sub-agent 1: Test Diagnostic (optional)
├── Sub-agent 2: Performance Profiler (optional)
└── Sub-agent 3: Release Manager (optional)
```

**Guidelines:**
- 1 primary agent = your AI lead
- 2-3 sub-agents maximum for context isolation
- Each sub-agent = single clear responsibility
- Each sub-agent has:
  - Very short system prompt
  - Limited context (subset of files, test report)
  - Structured short response (summary + plan)

**Context Pollution Reduction:**

Using isolated sub-agents for heavy operations (log analysis, multi-file scans) can reduce context pollution by **95-98%**[^19]. The sub-agent receives the heavy context, processes it, and returns only a summary to the main agent.

---

## The Iterative Workflow

### "Start Small, Expand Gradually"

This approach, recommended by the AI agent community[^20], emphasizes reactive rather than proactive complexity:

**The Process:**

1. **Start with a simple paragraph** describing clear intent
2. **Test and observe** the results
3. **Add complexity only if necessary** for specific failure modes
4. **Avoid preemptive structure**—don't anticipate all edge cases

**Visual Workflow:**

```
Objective defined
      ↓
Simple prompt (1-3 lines)
      ↓
Execute and observe
      ↓
      ├─→ Success → Capture pattern if repeatable
      ↓
   Errors?
      ↓
Add 1 targeted constraint
      ↓
Repeat from "Execute"
```

**Example Iteration:**

**Attempt 1 (Simple):**
```
"Add user authentication to the API"
```
→ Result: Model asks clarifying questions about JWT vs sessions

**Attempt 2 (Add context):**
```
"Add JWT-based user authentication to the FastAPI app
Context: We use SQLAlchemy for ORM, existing models in app/models/
Constraints: Use existing User model, don't modify database schema"
```
→ Result: Success

**Decision:** Since this pattern repeats, capture as skill:

```yaml
---
name: add-jwt-auth
description: Add JWT authentication to FastAPI apps
---

Use JWT-based auth with existing User model.
Don't modify database schema.
Ask if refresh tokens are needed.
```

---

## Measurement Over Dogma

### Key Metrics to Track

| Metric | Target | Why It Matters |
|:-------|:-------|:---------------|
| **First-attempt success rate** | >70% | Measures prompt clarity |
| **Average iteration count** | <2 | Indicates appropriate complexity |
| **Token cost per task** | Minimize | Efficiency indicator |
| **Response time** | Balance | Quality vs latency tradeoff |
| **Tool selection accuracy** | >80% | Toolkit appropriateness |

### Prompt Learning Results

Research from Arize AI demonstrates the power of optimizing only the system prompt[^21]:

**Results:**
- **+10% boost** on SWE Bench with targeted optimizations
- **+5% general coding performance** gains
- **Even larger improvements** when specialized to a single repository

**Key Insight:** Optimization without architectural changes—just better prompting—delivers measurable improvements.

### Claude Code Performance Guidelines

Temperature configuration for optimal performance[^22]:

| Temperature | Characteristic | Best For |
|:------------|:---------------|:---------|
| **0.0-0.2** | Very focused, deterministic | Code analysis, planning |
| **0.3-0.5** | Balanced with creativity | General development |
| **0.6-1.0** | More creative | Brainstorming |

**Critical Path Focus:**
- Launch Claude Code in critical directories (`api/`, `core/`)
- Avoid analyzing static assets or config files
- Apply systematic patterns to identify recurring inefficiencies

---

## Codebase Optimization for Agents

Research from Aaron Gustafson demonstrates measurable improvements from codebase optimization[^23]:

**Results:**
- **~40% reduction** in processing time
- **~75% reduction** in token usage
- **>80% reduction** in confusion and circular reasoning

**Applied Methods:**
- Consolidated, navigable documentation
- Quick validation scripts
- Explicit edge case handling
- Detailed module structure plans

**Key Quote:**
> "These improvements don't just help the AI agent. They help everyone."

**Implications:**
Optimizing your codebase for AI agents simultaneously improves it for human developers. The same clarity, structure, and explicitness benefits all collaborators.

---

## Claude Code & OpenCode Implementation

### Claude Code: Dev Lead Pattern

**Mental Model:** Use Claude Code as a "dev lead" with a simple loop, not as a rigid executor.

**Basic Prompts by Task Type:**

**Feature:**
```
"Ajoute la feature décrite dans ce ticket JIRA: ENG-4521, 
en respectant nos conventions FastAPI/SQLAlchemy 
et sans changer le schéma de la base."
```

**Bug:**
```
"Corrige ce bug: [error stacktrace], 
en modifiant le moins de code possible, 
en ajoutant un test unitaire."
```

**Refactor:**
```
"Refactor ce module pour utiliser notre nouvelle couche de service, 
sans changer les signatures publiques."
```

**Skills as Cheat Sheets:**

Structure skills as ultra-lightweight references:

```yaml
---
name: fastapi-crud-conventions
description: Follow our FastAPI CRUD patterns
---

## What I do
- Use `router.prefix` for resource routes
- Expose only `/items/{id}` and `/items/` endpoints
- Always validate with Pydantic V2 models
- Return 404 if not found, 422 for validation errors

## When to use me
Use for any new CRUD route or DTO.
Ask if pagination/search is required before implementing.
```

**Sub-agents for Context Management:**

Example sub-agents:
- `@test-diagnostic`: Receives test output, returns fix plan
- `@release-notes`: Generates changelog from PR list
- `@security-audit`: Reviews code for vulnerabilities

**MCP Setup (Minimal):**

```json
{
  "mcpServers": {
    "github": { "command": "gh", "args": ["mcp", "start"] },
    "sentry": { "command": "sentry-mcp-server" },
    "database": { "command": "postgres-mcp-server" }
  }
}
```

Keep to 3-5 essential integrations.

### OpenCode: Same Philosophy

**Configuration:**

```yaml
# .opencode/agents/dev.md
---
name: dev
description: Development agent with minimal toolkit
temperature: 0.3
maxSteps: 10
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Task
---

You are a development specialist.
Focus on minimal, working solutions.
Add complexity only when justified.
```

**Skills:**

OpenCode supports Claude-compatible skills:
- `.opencode/skills/<name>/SKILL.md`
- Or `~/.config/opencode/skills/<name>/SKILL.md`

```yaml
---
name: git-release
description: Draft release notes and bump version
---

## What I do
- Draft release notes from recent commits
- Bump version according to semver
- Create GitHub release with `gh release create`

## When to use me
Use when preparing a tagged release.
Ask about version bump type (major/minor/patch) if not specified.
```

**Plugin Selection (Minimalist):**

For "less is more" approach, keep only:
- 1 auth plugin (e.g., Antigravity)
- 1 context pruning plugin
- 1-2 business-specific plugins (testing, notifications)

### Double Mode Strategy

**[FINDING] Plan vs Build Mode Separation**

OpenCode's major innovation for safety and precision without heaviness is the separation of modes via distinct agents[^28]. This architectural pattern replaces complex "safety prompts" with structural separation.

**The Two Modes:**

| Mode | Permissions | Purpose | When to Use |
|:-----|:------------|:--------|:------------|
| **Plan Mode** (Tab: Plan) | Read-only | Analysis, strategy, verification | Before making changes |
| **Build Mode** (Tab: Build) | Read-write | Execution, implementation | After plan is validated |

**Why This Works:**

Traditional approaches use prompts for safety: "Don't delete files. Check before modifying."

The Double Mode approach enforces safety structurally:
- Plan agent **cannot** modify files, regardless of prompts
- Build agent **can** modify files, but only after explicit mode switch
- No amount of prompt engineering can bypass the permission boundary

**Workflow:**

```
1. Plan Mode: "Analyze @src/auth.ts. Propose a refactoring plan in bullet points."
   ↓
2. Human Review: Read the plan, validate mentally
   ↓
3. Build Mode: "Execute the plan. Use bash to verify tests after each change."
```

**Benefits:**

1. **Safety by Architecture:** Not by prompt—impossible to bypass
2. **Clear Thinking Phases:** Separation of analysis from action
3. **Human Validation Point:** Natural breakpoint for review
4. **Reduced Context Pollution:** Plan mode doesn't accumulate edit history

**Implementation with OpenCode:**

OpenCode formalizes this with separate tabs:
- **Plan Tab:** Read-only exploration, architecture decisions
- **Build Tab:** Read-write execution, file modifications

**Implementation with Claude Code:**

Simulate the pattern with permission modes:
```bash
# Start in plan mode (read-only)
claude --mode plan

# Review the plan, then switch to build mode
claude --mode build
```

Or use subagents:
```
Task: Analyze codebase for refactoring opportunities
Agent: Plan (read-only permissions)

Then:

Task: Execute the approved refactoring
Agent: general-purpose (read-write permissions)
```

**Comparison:**

| Approach | Safety Mechanism | Reliability |
|:---------|:-----------------|:------------|
| **Prompt-Based** | "Please don't delete files" | Low (can be ignored) |
| **Double Mode** | Structural permission separation | High (enforced by system) |

**Research Validation:**
This pattern aligns with the "Measure twice, cut once" philosophy formalized in Google's Antigravity IDE[^29]. It creates a procedural safety barrier that forces reflection before action.

---

## Red Flags: Over-Engineering Checklist

**Before Adding Complexity, Verify:**

- [ ] Can this task be solved with a simple prompt?
- [ ] Do I really need an agent, or would a workflow suffice?
- [ ] Is each tool used at least 80% of the time?
- [ ] Does context exceed 50% of the model window?
- [ ] Have I tested the simplified version before optimizing?
- [ ] Am I adding structure for hypothetical edge cases?
- [ ] Is this complexity solving a real problem or imagined ones?
- [ ] Can the model's built-in reasoning handle this instead of explicit instructions?

**Warning Signs of Over-Engineering:**

| Symptom | Likely Cause | Solution |
|:--------|:-------------|:---------|
| Agent seems paralyzed | Over-constraining | Remove rigid steps, provide objectives |
| Context always full | Bloated instructions | Move persistent rules to CLAUDE.md |
| Can't find information | Scattered references | Centralize in SKILL.md |
| Descriptions are verbose | Spoiling content | Use What-When-Not pattern |
| Using "magic syntax" | Over-structuring | Use natural language |
| Instructions too abstract | Over-generalization | Add concrete examples |

---

## Complete Examples

### Example 1: Before and After (Skill)

**❌ Before (Over-Engineered):**

```yaml
---
name: pdf-processor
description: This skill helps you process PDF files by reading them, extracting text, handling errors, and saving output to various formats. It uses pdfplumber library and implements best practices for PDF text extraction including handling multi-column layouts, headers, footers, and metadata extraction.
---

# PDF Processor

## Overview
This skill provides comprehensive PDF processing capabilities for the project.

## Installation
First, ensure pdfplumber is installed:
```bash
pip install pdfplumber
```

## Usage Instructions

### Step 1: Import Required Libraries
```python
import pdfplumber
import os
```

### Step 2: Open the PDF File
```python
with pdfplumber.open('file.pdf') as pdf:
    # Process PDF
```

### Step 3: Extract Text
```python
text = ""
for page in pdf.pages:
    text += page.extract_text()
```

### Step 4: Handle Errors
Always wrap in try-except blocks...

[Continues for 200+ lines]
```

**✅ After (Minimal):**

```yaml
---
name: processing-pdfs
description: Extracts text from PDF files. Use when working with PDF documents or need text extraction from PDFs. Not for image-based PDFs (use OCR tools).
---

## What I do
- Extract text from PDF files using pdfplumber
- Handle multi-page documents
- Preserve basic formatting

## Example
```python
import pdfplumber

with pdfplumber.open('document.pdf') as pdf:
    text = "\n".join(page.extract_text() for page in pdf.pages)
```

## When to use me
Use for text extraction from standard PDFs.
Ask if the PDF is scanned/image-based (requires OCR).
```

### Example 2: Before and After (Command)

**❌ Before (200+ lines of branching logic):**

```yaml
---
description: Complex interview command with predetermined paths
---

# Interview Workflow

## Step 1: Determine Project Type
IF user mentions web application:
  Ask about frontend framework
  IF React:
    Ask about state management
  IF Vue:
    Ask about composition API
ELSE IF user mentions API:
  Ask about authentication method
  [Continues with nested branches...]

## Step 2: Database Questions
IF database needed:
  Ask about SQL vs NoSQL
  [More branches...]

[200+ lines of predetermined logic]
```

**✅ After (22 lines):**

```yaml
---
description: Interview user in-depth to create a detailed spec. Use when requirements are unclear. Not for tasks with rigid output formats.
allowed-tools: AskUserQuestion, Write, Read, Glob
---

Follow user's instructions and use AskUserQuestionTool to interview them in depth.

When asking questions:
- Ask exactly one question at a time
- Offer clear, meaningful choices plus "Something else"
- Design options to help orient the user
- Never ask questions you could answer by reading files

Continue until you have enough detail to write a thorough specification.
```

### Example 3: Real-World Workflow

**Task:** Implement a new feature in a FastAPI application

**Traditional Approach (Complex):**
```
1. Create detailed specification (500 lines)
2. Define all edge cases upfront
3. Create multi-agent workflow:
   - Architect agent designs structure
   - Implementer agent writes code
   - Tester agent writes tests
   - Reviewer agent checks quality
4. Orchestrate with complex state management
```

**"Less is More" Approach:**
```
User: "Add user profile endpoint with avatar upload following our existing patterns"

Agent: 
- Reads existing user-related endpoints for patterns
- Implements /users/{id}/profile with avatar
- Asks: "Should we validate avatar size/format?"
- User: "Yes, max 2MB, jpg/png only"
- Agent adds validation
- Tests: "pytest passes, ready to review"
```

**Why the simple approach wins:**
- Faster to execute
- Adapts to actual requirements revealed during work
- Uses existing patterns from codebase
- Defers edge cases until relevant

---

## References

### Research Papers & Articles

[^1]: Joshua Berkowitz, "Solving Tool Overload in AI Agents with Semantic Selection" (2024). https://joshuaberkowitz.us/blog/news-1/solving-tool-overload-in-ai-agents-with-semantic-selection-1735

[^2]: Chroma Research Team, "Context Rot: Why LLMs Get Distracted and How to Write Shorter Prompts" (2024). https://blog.promptlayer.com/why-llms-get-distracted-and-how-to-write-shorter-prompts

[^3]: Aakash Gupta, "I Studied 1,500 Academic Papers on Prompt Engineering" (2024). https://aakashgupta.medium.com/i-studied-1-500-academic-papers-on-prompt-engineering-heres-why-everything-you-know-is-wrong-2fc97c1735a7

[^4]: Boris Cherny and Anthropic Team, Claude Code Best Practices (2026). https://www.anthropic.com/engineering/claude-code-best-practices

[^5]: Anthropic Research Team, "Building Effective Agents" (2024). https://www.anthropic.com/research/building-effective-agents

[^6]: See reference [2]

[^7]: See reference [1]

[^8]: MCP Server Efficiency Presentation (2024). https://www.youtube.com/watch?v=eOMq4suzl3U

[^9]: See reference [1]

[^10]: Raphael Mansuy, LinkedIn post on tool retrieval benchmarking (2024). https://www.linkedin.com/posts/raphaelmansuy_benchmarking-tool-retrieval-for-large-language-activity-7302570464206282752-EIlu

[^11]: Reddit r/AI_Agents, "Are we overengineering agents when simple systems work?" (2024). https://www.reddit.com/r/AI_Agents/comments/1ph5m9c/are_we_overengineering_agents_when_simple_systems/

[^12]: See reference [3]

[^13]: Richard Oleson, LinkedIn post on engineering prompts for LLMs (2024). https://www.linkedin.com/posts/richard-oleson-408784101_working-with-engineering-prompts-for-llm-activity-7386060970370060288-CcAU

[^14]: Anthropic Engineering, "Effective Context Engineering for AI Agents" (2024). https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

[^15]: See reference [10]

[^16]: See reference [8]

[^17]: See reference [8]

[^18]: Based on actual implementation in .claude/commands/reviewer-v2.md

[^19]: Ethan Houseworth and community contributors, techniques for context isolation (2024)

[^20]: SparkCo AI, "Mastering Agent Prompt Engineering: A 2025 Deep Dive" (2025). https://sparkco.ai/blog/mastering-agent-prompt-engineering-a-2025-deep-dive

[^21]: Arize AI, "Claude MD Best Practices: Learned from Optimizing Claude Code with Prompt Learning" (2024). https://arize.com/blog/claude-md-best-practices-learned-from-optimizing-claude-code-with-prompt-learning/

[^22]: Claude Code Performance Documentation (2024). https://claude.com/blog/optimize-code-performance-quickly

[^23]: Aaron Gustafson, "Optimizing Your Codebase for AI Coding Agents" (2024). https://dev.to/aarongustafson/optimizing-your-codebase-for-ai-coding-agents-4ndm

[^24]: Lita Research Team, "Lita: Light Agent Uncovers the Agentic Coding Capabilities of LLMs" (2025). https://arxiv.org/abs/2509.25873

[^25]: LIMI Research Team, "Less is More for Agency" (2025). https://arxiv.org/pdf/2509.17567

[^26]: Chain of Draft Research, "Chain of Draft: Concise Prompting Reduces LLM Costs" (2025). https://ajithp.com/2025/03/02/chain-of-draft-llm-prompting/

[^27]: Anthropic Engineering, "Effective Context Engineering for AI Agents" (2024). https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

[^28]: OpenCode Documentation, "The definitive guide to OpenCode" (2025). https://jpcaparas.medium.com/the-definitive-guide-to-opencode-from-first-install-to-production-workflows-aae1e95855fb

[^29]: Google Antigravity Documentation, "Planning vs Fast Mode" (2025). https://dev.to/this-is-learning/planning-vs-fast-mode-in-google-antigravity-when-and-how-to-use-each-36n5

### Additional Resources

- **Anthropic Prompt Improver**: https://anthropic.mintlify.app/en/docs/build-with-claude/prompt-engineering/prompt-improver
- **Claude Code Documentation**: https://code.claude.com/docs
- **OpenCode Documentation**: https://opencode.ai/docs
- **Agent Skills Standard**: https://agentskills.io/specification
- **HyphaDev Blog**: https://www.hyphadev.io/blog/ai-prompt-optimizer
- **CURAM AI**: https://curam-ai.com.au/the-role-of-ai-agents-efficiency-through-minimalism-and-orchestration/

---

## Conclusion

The optimization of AI agents is not a race toward complexity but a pursuit of **elegant simplicity**. Research consistently demonstrates that:

1. **Simpler prompts outperform complex ones** for modern models
2. **Tool minimalism** (3-5 essential tools) beats comprehensive tool loading
3. **Context budget thinking** maximizes signal-to-noise ratio
4. **Reactive constraints** work better than proactive over-constraint
5. **Measurement** should drive decisions, not architectural sophistication

**Your New Workflow:**

1. **Start simple**: 1-3 line prompts, minimal structure
2. **Complexify only if necessary**: Add constraints after observing failures
3. **Measure before optimizing**: Track completion rates, not complexity
4. **Prefer composition to complexity**: Reusable, focused components

By adopting this approach, you'll create AI agent systems that are more performant, reliable, and economical—regardless of the tool you choose (Claude Code, OpenCode, or others).

**Final Principle:**

> "The best abstraction is the one you didn't need to create."

Start minimal. Add only what proves necessary. Trust the model's intelligence. Measure everything.

---

*Document Version: 1.0*  
*Last Updated: 2026-02-04*  
*Based on research through February 2026*
