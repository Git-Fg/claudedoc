# claudedoc

## Docs QA ✅

Quick checks to run after documentation edits:

- Search for inconsistent phrases:
  - `grep -R "community extension" -n .`
  - `grep -R "runs inline" -n .`
  - `grep -R "recursion loophole" -n .`
- Ensure:
  - `license` is documented as part of the Agent Skills standard where applicable
  - All `context: fork` mentions indicate subagent spawn semantics (or clearly note experimental behavior)
  - References to `agentskills.io` are included when discussing standard fields

Add items here as the QA checklist grows.
