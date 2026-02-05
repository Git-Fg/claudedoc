---
title: Less is More - Quick Reference
audience: all
last-reviewed: 2026-02-04
---

# Quick Reference: Less is More Patterns

One-page cheat sheet for minimalist AI agent design. For complete methodology and research, see [07-less-is-more.md](07-less-is-more.md).

---

## Decision Flowchart

```
New Task
    ↓
Is output format strictly predefined?
    ↓
    ├─→ YES → Is it safety-critical?
    │       ↓
    │       ├─→ YES → Use STRUCTURED PROMPT
    │       └─→ NO  → Use STRUCTURED PROMPT
    ↓
    └─→ NO → Start with SIMPLE PROMPT (1-3 lines)
        ↓
    Execute and Observe
        ↓
        ├─→ Success → Done (capture if repeatable)
        ↓
    Errors?
        ↓
    Add 1 targeted constraint
        ↓
    Repeat
```

---

## 7 Minimalist Prompt Templates

### Template 1: Direct Intent (1 line)
**Best for:** Simple, well-understood tasks

```
[Action] [object] [context]

Examples:
- "Calcule la moyenne d'une liste de nombres en Python"
- "Find all TODO comments in the codebase"
- "Refactor this function to use async/await"
```

### Template 2: Intent + Context + Constraints (3 lines)
**Best for:** Tasks requiring background knowledge and boundaries

```
Goal: [What you want]
Context: [Relevant background]
Constraints: [Key limitations]

Example:
Goal: Ajoute l'endpoint POST /users avec validation email/password
Context: API REST FastAPI avec SQLAlchemy, repo dans ./app
Constraints: Ne touche pas aux migrations, utilise les modèles existants
```

### Template 3: Objective + Success Criteria (2-3 lines)
**Best for:** Complex tasks needing clear completion definition

```
Objective: [High-level goal]
Success Criteria: [Measurable outcomes]

Example:
Objective: Implement JWT authentication endpoint
Success Criteria:
- Returns token on valid credentials (200)
- Returns 401 on invalid credentials
- Follows existing auth patterns
```

### Template 4: Natural Language Delegation (1 paragraph)
**Best for:** Tasks requiring autonomy and adaptation

```
[Action needed]. [Context/resource to consult]. [Success criteria/constraint].

Example:
"Refactor this authentication module to use the new security patterns. 
Handle any breaking changes by checking the migration guide first. 
Ensure all existing tests still pass."
```

### Template 5: Mission Control Pattern (XML structure)
**Best for:** Complex missions requiring explicit criteria

```markdown
<mission_control>
<objective>[What to achieve]</objective>
<success_criteria>[How to know it's done]</success_criteria>
</mission_control>

Example:
<mission_control>
<objective>Create a skill for processing PDFs</objective>
<success_criteria>Skill extracts text and handles errors gracefully</success_criteria>
</mission_control>
```

### Template 6: Chain of Draft (Dense Reasoning)
**Best for:** Complex reasoning tasks requiring speed and efficiency

**The Instruction:**
```
"Use Chain of Draft: dense reasoning, maximum 5 words per step."
```

**Example Output:**
```
Traditional: "First, I need to check the authentication module. 
The auth.ts file appears to use JWT tokens..."

Chain of Draft: "Check auth.ts → JWT found → Expiry check missing."
```

**Benefits:**
- 80% reduction in token usage
- 50-75% latency reduction
- Maintains or improves accuracy
- Forces focus on logic, not syntax

### Template 7: Canonical Examples (Pattern Matching)
**Best for:** Establishing coding standards and conventions

**Structure:**
```markdown
Follow the pattern in @src/utils/canonical-example.ts:
- Use Result<T, E> for error handling
- Validate early, return early
- Be exhaustive in type checking

See the file for the definitive implementation.
```

**Principle:**
One perfect code example > 1000 lines of abstract rules

**Research:**
78 curated examples outperformed 10,000 generic samples by 53.7% (LIMI study)

---

## Skill Template (Minimal)

```yaml
---
name: [gerund-form-name]
description: [What it does]. Use when [trigger conditions]. Not for [exclusions].
---

## What I do
- [Capability 1]
- [Capability 2]
- [Capability 3]

## When to use me
[Trigger conditions and exclusions].
[Optional: Ask about X before implementing Y].
```

**Example:**

```yaml
---
name: processing-pdfs
description: Extracts text from PDF files. Use when working with PDF documents. Not for image-based PDFs (use OCR tools).
---

## What I do
- Extract text from PDF files using pdfplumber
- Handle multi-page documents
- Preserve basic formatting

## When to use me
Use for text extraction from standard PDFs.
Ask if the PDF is scanned/image-based (requires OCR).
```

---

## Command Template (Minimal)

```yaml
---
description: [What it does]. Use when [trigger conditions]. Not for [exclusions].
allowed-tools: [Essential tools only - 3-5 max]
---

[Objective statement]

[Guidelines - 3-5 bullet points maximum]

[When to stop/complete]
```

**Example (22-line interview command):**

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

---

## Context Management

### Context Pruning & Goldilocks Zone

**The Goal:** Find the "just right" amount of context—not too little, not too much.

| Zone | Context Amount | Result |
|:-----|:---------------|:-------|
| **Too Little** | <5% of capacity | Hallucinations due to insufficient info |
| **Goldilocks** | 20-50% of capacity | Optimal signal-to-noise ratio |
| **Too Much** | >70% of capacity | Confusion and degraded performance |

**Pruning Checklist:**
- [ ] Remove obsolete documentation
- [ ] Delete irrelevant past conversations
- [ ] Use `/reset` frequently to start fresh
- [ ] Keep only one topic per session

**Signs You Need Pruning:**
- Agent repeats questions you already answered
- Responses become generic or off-topic
- Latency increases significantly

### Just-in-Time Context

**Principle:** Don't preload context—let the agent explore.

**Comparison:**

| Approach | Method | Result |
|:---------|:-------|:-------|
| **"Thick Agent"** | "Here's the entire /src folder" | Context polluted with irrelevant files |
| **"Thin Agent"** | "Here are grep and cat. Explore to find the bug" | Only relevant context loaded |

**Implementation:**
```
❌ Don't: "Read these 20 files [attached]"

✅ Do: "Find the bug in the authentication system"
   → Agent uses tools to explore
   → Agent reads only auth.ts, user.ts, jwt.ts
   → Context stays focused
```

---

## Double Mode Strategy

**Principle:** Separate analysis from execution.

| Mode | Permissions | Purpose |
|:-----|:------------|:--------|
| **Plan Mode** | Read-only | Analysis, strategy, verification |
| **Build Mode** | Read-write | Execution, implementation |

**Workflow:**
```
1. Plan Mode: "Analyze @src/auth.ts. Propose a refactoring plan."
   ↓
2. Human Review: Read and validate the plan
   ↓
3. Build Mode: "Execute the plan. Verify tests pass."
```

**Why It Works:**
- Safety by architecture (not by prompt)
- Natural validation breakpoint
- Clear thinking phases
- Reduces context pollution

**OpenCode Implementation:**
- Use **Plan Tab** for read-only analysis
- Use **Build Tab** for read-write execution

---

## Red Flags Checklist

**Before Adding Complexity:**

- [ ] Can this be solved with a simple prompt? (1-3 lines)
- [ ] Do I really need an agent, or would a workflow suffice?
- [ ] Is each tool used at least 80% of the time?
- [ ] Does my toolkit exceed 10 tools? (Precision drops beyond this)
- [ ] Have I tested the simple version first?
- [ ] Am I adding structure for hypothetical edge cases?
- [ ] Can the model's built-in reasoning handle this instead?

**Warning Signs:**

| Symptom | Problem | Fix |
|:--------|:--------|:----|
| Agent paralyzed | Over-constraining | Provide objectives, not procedures |
| Context always full | Bloated instructions | Centralize rules in CLAUDE.md |
| Can't find info | Scattered references | Put core knowledge in SKILL.md |
| Verbose descriptions | Spoiling content | Use What-When-Not pattern |
| "Magic syntax" | Over-structuring | Use natural language |
| Instructions too abstract | Over-generalization | Add concrete examples |

---

## When Simplicity Wins vs Structure Needed

### Use SIMPLE Prompts When:
- ✓ Discovery and exploration tasks
- ✓ Requirements gathering
- ✓ Code review and analysis
- ✓ One-off coding tasks
- ✓ Brainstorming and ideation
- ✓ Bug investigation

### Use STRUCTURED Prompts When:
- ✓ Repeatable workflows with predefined outputs
- ✓ System integrations (CRMs, support platforms)
- ✓ Production-critical systems
- ✓ Explicit chain-of-thought required
- ✓ Safety-critical systems
- ✓ Regulatory compliance

---

## Toolkit Focus Guidelines

**The 10-Tool Maximum Rule:**
Beyond 10 tools, precision declines significantly.

**Recommended Hierarchy:**

| Tier | Count | Loading Strategy |
|:-----|:------|:-----------------|
| **Essential** | 3-5 tools | Statically loaded |
| **Specialized** | 2-3 tools | Per workflow |
| **Contextual** | Variable | Dynamic selection |

**Tool Selection Strategy:**

```typescript
// ❌ Don't: Load 20+ tools statically
const tools = [tool1, tool2, ..., tool20];

// ✅ Do: Select 3-5 relevant tools dynamically
const relevantTools = await selectToolsForTask(userQuery);
```

---

## Component Size Targets

| Component | Target Size | Maximum |
|:----------|:------------|:--------|
| **Simple Skill** | 20-50 lines | 100 lines |
| **Complex Skill** | 100-200 lines | 500 lines |
| **Command** | 10-30 lines | 50 lines |
| **Agent Prompt** | 5-15 lines | 30 lines |

**Remember:** Skills compound—each new skill makes all future work faster. But bloated skills hurt performance. Start minimal.

---

## The Iterative Workflow (Visual)

```
┌─────────────────┐
│  Define Goal    │
└────────┬────────┘
         ↓
┌─────────────────┐
│ Simple Prompt   │ ← 1-3 lines
│ (1-3 lines)     │
└────────┬────────┘
         ↓
┌─────────────────┐
│   Execute       │
└────────┬────────┘
         ↓
    ┌────┴────┐
    ↓         ↓
Success    Errors?
    ↓         ↓
   Done  ┌────┴────┐
         ↓         ↓
    Add 1   Still fails?
    line         ↓
    only    Repeat
         ↓         
    ┌────┘
    ↓
Capture as
Skill/Command
(if repeatable)
```

---

## Key Metrics to Track

| Metric | Target | Why |
|:-------|:-------|:----|
| First-attempt success | >70% | Prompt clarity |
| Average iterations | <2 | Appropriate complexity |
| Token cost per task | Minimize | Efficiency |
| Tool selection accuracy | >80% | Toolkit appropriateness |

---

## Quick Comparison: Simple vs Complex

### Prompt Example

**Complex (Bad):**
```javascript
"Écris une fonction Python qui calcule la moyenne d'une liste de nombres, 
en utilisant une approche fonctionnelle avec des lambda, 
en gérant les cas d'erreur, avec une documentation complète, 
en suivant PEP8, et en retournant None si la liste est vide"
```

**Simple (Good):**
```javascript
"Calcule la moyenne d'une liste de nombres en Python"
```

### Skill Example

**Complex (Bad - 200+ lines):**
```markdown
# PDF Processor

## Overview
Comprehensive PDF processing capabilities...

## Installation
First, ensure pdfplumber is installed...

## Usage Instructions

### Step 1: Import Required Libraries
[10 steps... 200 lines total]
```

**Simple (Good - ~20 lines):**
```yaml
---
name: processing-pdfs
description: Extracts text from PDF files. Use when working with PDF documents.
---

## What I do
- Extract text from PDF files using pdfplumber
- Handle multi-page documents

## When to use me
Use for text extraction from standard PDFs.
```

---

## Remember

> "The best abstraction is the one you didn't need to create."

1. **Start simple**: 1-3 line prompts
2. **Complexify only if necessary**: Add constraints after failures
3. **Measure before optimizing**: Data > opinions
4. **Prefer composition**: Reusable, focused components

---

## Reference

For complete methodology, research citations, and implementation examples:

📄 **[07-less-is-more.md](07-less-is-more.md)** - Comprehensive Guide to AI Agent Minimalism

---

*Version: 1.1*  
*Last Updated: 2026-02-05*  
*Added: Agent Complexity Law, LIMI, Chain of Draft, Double Mode Strategy*
