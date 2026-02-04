# The Complete Guide to Building Skills for Claude

> **Warning**: This document contains a mix of official Anthropic documentation and custom thecattoolkit_v3 extensions. Claims marked as official have been verified against https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices. Custom extensions are not part of the official Anthropic specification.

---

## Contents

1. [Introduction](#introduction) — Page 3
2. [Fundamentals](#chapter-1-fundamentals) — Page 4
3. [Planning and design](#chapter-2-planning-and-design) — Page 7
4. [Testing and iteration](#chapter-3-testing-and-iteration) — Page 14
5. [Distribution and sharing](#chapter-4-distribution-and-sharing) — Page 18
6. [Patterns and troubleshooting](#chapter-5-patterns-and-troubleshooting) — Page 21
7. [Resources and references](#chapter-6-resources-and-references) — Page 28

---

## Introduction

A **skill** is a set of instructions — packaged as a simple folder — that teaches Claude how to handle specific tasks or workflows. Skills are one of the most powerful ways to customize Claude for your specific needs. Instead of re-explaining your preferences, processes, and domain expertise in every conversation, skills let you teach Claude once and benefit every time.

Skills are powerful when you have repeatable workflows: generating frontend designs from specs, conducting research with consistent methodology, creating documents that follow your team's style guide, or orchestrating multi-step processes. They work well with Claude's built-in capabilities like code execution and document creation. For those building MCP integrations, skills add another powerful layer helping turn raw tool access into reliable, optimized workflows.

This guide covers everything you need to know to build effective skills — from planning and structure to testing and distribution. Whether you're building a skill for yourself, your team, or for the community, you'll find practical patterns and real-world examples throughout.

### What you'll learn:
* Technical requirements and best practices for skill structure
* Patterns for standalone skills and MCP-enhanced workflows
* Patterns we've seen work well across different use cases
* How to test, iterate, and distribute your skills

### Who this is for:
* Developers who want Claude to follow specific workflows consistently
* Power users who want Claude to follow specific workflows
* Teams looking to standardize how Claude works across their organization

### Two Paths Through This Guide
* **Building standalone skills?** Focus on Fundamentals, Planning and Design, and categories 1-2.
* **Enhancing an MCP integration?** The "Skills + MCP" section and category 3 are for you.
* Both paths share the same technical requirements, but you choose what's relevant to your use case.

**What you'll get out of this guide:** By the end, you'll be able to build a functional skill in a single sitting. Expect about 15-30 minutes to build and test your first working skill using the skill-creator.

---

## Chapter 1: Fundamentals

### What is a skill?
A skill is a folder containing:
* **SKILL.md** (required): Instructions in Markdown with YAML frontmatter.
* **scripts/** (optional): Executable code (Python, Bash, etc.).
* **references/** (optional): Documentation loaded as needed.
* **assets/** (optional): Templates, fonts, icons used in output.

### Core design principles

#### Progressive Disclosure
Skills use a three-level system to minimize token usage while maintaining specialized expertise:
1. **First level (YAML frontmatter):** Always loaded in Claude's system prompt. Provides just enough information for Claude to know when each skill should be used without loading all of it into context.
2. **Second level (SKILL.md body):** Loaded when Claude thinks the skill is relevant to the current task. Contains the full instructions and guidance.
3. **Third level (Linked files):** Additional files bundled within the skill directory that Claude can choose to navigate and discover only as needed.

#### Composability
Claude can load multiple skills simultaneously. Your skill should work well alongside others and not assume it's the only capability available.

#### Portability
Skills work identically across Claude.ai, Claude Code, and API. Create a skill once and it works across all surfaces without modification, provided the environment supports any dependencies the skill requires.

---

### For MCP Builders: Skills + Connectors
> 💡 *Building standalone skills without MCP? Skip to Planning and Design — you can always return here later.*

If you already have a working MCP server, skills are the knowledge layer on top — capturing the workflows and best practices so Claude can apply them consistently.

#### The kitchen analogy
* **MCP** provides the **professional kitchen**: access to tools, ingredients, and equipment.
* **Skills** provide the **recipes**: step-by-step instructions on how to create something valuable.

Together, they enable users to accomplish complex tasks without needing to figure out every step themselves.

#### How they work together:
| MCP (Connectivity) | Skills (Knowledge) |
| :--- | :--- |
| Connects Claude to your service (Notion, Asana, Linear, etc.) | Teaches Claude how to use your service effectively |
| Provides real-time data access and tool invocation | Captures workflows and best practices |
| **What Claude can do** | **How Claude should do it** |

#### Why this matters for your MCP users
**Without skills:**
* Users connect your MCP but don't know what to do next.
* Support tickets asking "how do I do X with your integration."
* Each conversation starts from scratch.
* Inconsistent results because users prompt differently each time.
* Users blame your connector when the real issue is workflow guidance.

**With skills:**
* Pre-built workflows activate automatically when needed.
* Consistent, reliable tool usage.
* Best practices embedded in every interaction.
* Lower learning curve for your integration.

---

## Chapter 2: Planning and Design

### Start with use cases
Before writing any code, identify 2-3 concrete use cases your skill should enable.

**Good use case definition:**
* **Use Case:** Project Sprint Planning
* **Trigger:** User says "help me plan this sprint" or "create sprint tasks"
* **Steps:**
    1. Fetch current project status from Linear (via MCP)
    2. Analyze team velocity and capacity
    3. Suggest task prioritization
    4. Create tasks in Linear with proper labels and estimates
* **Result:** Fully planned sprint with tasks created

**Ask yourself:**
* What does a user want to accomplish?
* What multi-step workflows does this require?
* Which tools are needed (built-in or MCP)?
* What domain knowledge or best practices should be embedded?

---

### Common skill use case categories

#### Category 1: Document & Asset Creation
* **Used for:** Creating consistent, high-quality output including documents, presentations, apps, designs, code, etc.
* **Real example:** `frontend-design` skill.
* **Description:** "Create distinctive, production-grade frontend interfaces with high design quality. Use when building web components, pages, artifacts, posters, or applications."
* **Key techniques:** Style guides, template structures, quality checklists.

#### Category 2: Workflow Automation
* **Used for:** Multi-step processes that benefit from consistent methodology, including coordination across multiple MCP servers.
* **Real example:** `skill-creator` skill.
* **Description:** "Interactive guide for creating new skills. Walks the user through use case definition, frontmatter generation, instruction writing, and validation."
* **Key techniques:** Validation gates, iterative refinement loops.

#### Category 3: MCP Enhancement
* **Used for:** Workflow guidance to enhance the tool access an MCP server provides.
* **Real example:** `sentry-code-review` skill.
* **Description:** "Automatically analyzes and fixes detected bugs in GitHub Pull Requests using Sentry's error monitoring data via their MCP server."
* **Key techniques:** Coordinating multiple MCP calls, error handling for common MCP issues.

---

### Define success criteria
Aim for rigor but accept an element of "vibes-based" assessment.

#### Quantitative metrics:
* **Skill triggers on 90% of relevant queries:** Track automatic loads vs. explicit invocation.
* **Completes workflow in X tool calls:** Compare task efficiency with and without the skill.
* **0 failed API calls per workflow:** Monitor MCP server logs for retry rates.

#### Qualitative metrics:
* **Users don't need to prompt Claude about next steps:** Note how often you need to redirect.
* **Workflows complete without user correction:** Compare outputs for structural consistency.
* **Consistent results across sessions:** Can a new user accomplish the task on the first try?

---

### Technical Requirements

#### File structure
```text
your-skill-name/
├── SKILL.md              # Required - main skill file
├── scripts/              # Optional - executable code
│   ├── process_data.py   # Example
│   └── validate.sh       # Example
├── references/           # Optional - documentation
│   ├── api-guide.md      # Example
│   └── examples/         # Example
└── assets/               # Optional - templates, etc.
    └── report-template.md # Example
```

#### Critical rules
* **SKILL.md naming:** Must be exactly `SKILL.md` (case-sensitive).
* **Skill folder naming:** Use `kebab-case` (e.g., `notion-project-setup`). No spaces, underscores, or capitals.
* **No README.md:** Do not include a `README.md` *inside* the skill folder. All documentation goes in `SKILL.md` or `references/`.

#### YAML frontmatter: The most important part
This is how Claude decides whether to load your skill.

**Minimal required format:**
```yaml
---
name: your-skill-name
description: What it does. Use when user asks to [specific phrases].
---
```

**Field requirements:**
* **name:** kebab-case only, should match folder name. Maximum 64 characters. Cannot contain reserved words ("anthropic", "claude").
* **description:** MUST include both **What** the skill does and **When** to use it. Under 1024 characters. No XML tags (`<` or `>`). Always write in third person.

**Naming recommendations:**
* Use **gerund form** (verb + -ing): `processing-pdfs`, `analyzing-spreadsheets`, `managing-databases`

#### Optional fields:
* **context:** Run skill in isolated context
* **agent:** Which subagent type to use
* **user-invocable:** Visibility in / menu
* **disable-model-invocation:** Require explicit user intent
* **argument-hint:** Hint for autocomplete
* **model:** Model override
* **hooks:** Event-driven automation

> **Note**: The Agent Skills standard requires `name` and `description` and also recognizes `license` as a schema field (see https://agentskills.io/specification). Fields such as `context`, `agent`, `user-invocable`, and `disable-model-invocation` are Claude Code extensions not part of the base Agent Skills Standard. Some fields (e.g., `allowed-tools`) are experimental and may vary across implementations.

#### Security restrictions:
* Forbidden to use "claude" or "anthropic" in skill names
* Cannot contain XML angle brackets (`<` or `>`) in name field

---

### Writing effective skills

#### The description field structure:
`[What it does] + [When to use it] + [Key capabilities]`

**Example of a good description:**
> `description: Analyzes Figma design files and generates developer handoff documentation. Use when user uploads .fig files, asks for "design specs", "component documentation", or "design-to-code handoff".`

#### Writing the main instructions
After the frontmatter, use Markdown.

**Recommended structure:**
```markdown
---
name: your-skill
description: [...]
---

# Your Skill Name

## Instructions

### Step 1: [First Major Step]
Clear explanation of what happens.

Example:
```bash
python scripts/fetch_data.py --project-id PROJECT_ID
```
Expected output: [describe success]

## Examples
### Example 1: [Common scenario]
User says: "Set up a marketing campaign"
Actions: [Steps]
Result: [Confirmation]

## Troubleshooting
Error: [Message]
Cause: [Why]
Solution: [How to fix]
```

---

### Best Practices for Instructions
* **Be Specific and Actionable:** Instead of "Validate the data," use "Run `python scripts/validate.py --input {filename}`."
* **Include error handling:** Provide steps for common issues like "MCP Connection Failed."
* **Reference bundled resources:** "Before writing queries, consult `references/api-patterns.md`."
* **Use progressive disclosure:** Keep `SKILL.md` focused on core instructions; move details to `references/`.

---

## Chapter 3: Testing and Iteration

### Testing Levels
1. **Manual testing in Claude.ai:** Run queries directly.
2. **Scripted testing in Claude Code:** Automate test cases for repeatable validation.
3. **Programmatic testing via skills API:** Build evaluation suites.

> **Pro Tip:** Iterate on a single challenging task until Claude succeeds, then extract that winning approach into the skill.

### Recommended Testing Approach

#### 1. Triggering tests
**Goal:** Ensure your skill loads at the right times.
* ✅ Triggers on obvious tasks.
* ✅ Triggers on paraphrased requests.
* ❌ Doesn't trigger on unrelated topics.

#### 2. Functional tests
**Goal:** Verify the skill produces correct outputs.
* Valid outputs generated.
* API calls succeed.
* Error handling works.

#### 3. Performance comparison
**Goal:** Prove the skill improves results vs. baseline (e.g., fewer messages, fewer tokens, 0 failed API calls).

---

### Using the `skill-creator` skill
Available in Claude.ai via plugin directory or Claude Code.
* **Creating:** Generates skills from natural language.
* **Reviewing:** Flags vague descriptions or missing triggers.
* **Iterative improvement:** Bring edge cases back to `skill-creator` to refine instructions.

**Iteration based on feedback:**
* **Undertriggering:** Add more detail/keywords to the description.
* **Overtriggering:** Add negative triggers ("Do NOT use for...").
* **Execution issues:** Improve instructions or add script-based validations.

---

## Chapter 4: Distribution and Sharing

### Distribution Model (January 2026)
* **Individuals:** Upload to Claude.ai via Settings > Capabilities > Skills or place in Claude Code skills directory.
* **Organizations:** Admins can deploy workspace-wide with automatic updates.

### An open standard
**Agent Skills** is an open standard. Skills should be portable across tools and platforms.

### Using skills via API
Programmatic use cases use the `/v1/skills` endpoint and the `container.skills` parameter in Messages API requests.
* **Note:** API skills require the **Code Execution Tool beta**.

| Use Case | Best Surface |
| :--- | :--- |
| End users interacting directly | Claude.ai / Claude Code |
| Manual testing/iteration | Claude.ai / Claude Code |
| Applications using skills programmatically | API |
| Production deployments at scale | API |

### Recommended approach today
1. **Host on GitHub:** Use a public repo with a clear `README.md` (separate from the skill folder).
2. **Document in Your MCP Repo:** Link to skills and explain the value.
3. **Create an Installation Guide:** Provide clear steps (Download -> Install -> Enable -> Test).

**Positioning your skill:** Focus on **outcomes**, not features.
* ✅ **Good:** "The ProjectHub skill enables teams to set up complete project workspaces in seconds..."
* ❌ **Bad:** "The ProjectHub skill is a folder containing YAML frontmatter..."

---

## Chapter 5: Patterns and Troubleshooting

### Choosing your approach
* **Problem-first:** "I need to set up a workspace" -> Skill orchestrates tools.
* **Tool-first:** "I have Notion connected" -> Skill provides expertise on how to use it.

### Common Patterns
1. **Sequential workflow orchestration:** multi-step processes in a specific order.
2. **Multi-MCP coordination:** Workflows spanning multiple services (e.g., Figma -> Drive -> Linear -> Slack).
3. **Iterative refinement:** Output quality improves with validation scripts and loops.
4. **Context-aware tool selection:** Same outcome, different tools depending on file type/size.
5. **Domain-specific intelligence:** Specialized knowledge (e.g., financial compliance checks) before action.

---

### Troubleshooting Guide

#### Skill won't upload
* **Error:** "Could not find SKILL.md" -> **Fix:** Rename file exactly (case-sensitive).
* **Error:** "Invalid frontmatter" -> **Fix:** Check YAML delimiters `---` and quotes.
* **Error:** "Invalid skill name" -> **Fix:** Use kebab-case (no capitals/spaces).

#### Skill doesn't trigger
* **Fix:** Revise description. Avoid being generic. Include phrases users actually say. Use the debugging command: "When would you use the [skill name] skill?"

#### Instructions not followed
* **Causes:** Instructions too verbose (use bullets), instructions buried (move to top), or ambiguous language.
* **Model "laziness":** Add explicit encouragement ("Take your time," "Do not skip validation").

#### Large context issues
* **Symptom:** Slow responses.
* **Solution:** Keep `SKILL.md` under 500 lines for optimal performance. Move docs to `references/`.

---

## Chapter 6: Resources and References

### Official Documentation
* Best Practices Guide
* Skills Documentation
* API Reference
* MCP Documentation

### Example skills
* **GitHub:** `anthropics/skills` (Contains Anthropic-created skills you can customize).

### Getting Support
* **Technical Questions:** Community forums at the Claude Developers Discord.
* **Bug Reports:** GitHub Issues: `anthropics/skills/issues`.

---

## Reference A: Quick Checklist

### Before you start
- [ ] Identified 2-3 concrete use cases
- [ ] Tools identified (built-in or MCP)
- [ ] Planned folder structure

### During development
- [ ] Folder named in kebab-case
- [ ] `SKILL.md` file exists (exact spelling)
- [ ] YAML frontmatter has `---` delimiters
- [ ] description includes WHAT and WHEN
- [ ] No XML tags (`< >`) anywhere
- [ ] Instructions are clear and actionable
- [ ] Error handling included
- [ ] Examples provided

### Before upload
- [ ] Tested triggering on obvious and paraphrased tasks
- [ ] Functional tests pass
- [ ] Compressed as `.zip` file

---

## Reference B: YAML frontmatter

**Required Fields:**
```yaml
---
name: skill-name-in-kebab-case
description: What it does and when to use it. Include specific trigger phrases.
---
```

**All Optional Fields:**
```yaml
name: skill-name
description: [required description]
license: MIT
allowed-tools: "Bash(python:*) Bash(npm:*) WebFetch"
metadata:
  author: Company Name
  version: 1.0.0
  mcp-server: server-name
  category: productivity
  tags: [automation]
documentation: https://example.com/docs
support: support@example.com
```

---

## Reference C: Complete skill examples
* **Document Skills:** PDF, DOCX, PPTX, XLSX creation.
* **Partner Skills Directory:** View skills from partners like Asana, Atlassian, Canva, Figma, Sentry, and Zapier.
