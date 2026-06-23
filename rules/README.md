# Rules

Path- or context-scoped constraints that enforce quality automatically.

A rule applies without being asked: when the AI touches matching files or works in a matching
context, the rule's constraints take effect. Rules turn a quality standard into a guardrail rather
than a suggestion.

## What belongs here

- Constraints scoped to a language, framework, file pattern, or task type.
- The conditions under which each rule should auto-apply.

## Conventions

- One rule per file, named for the constraint it enforces (`type-driven-values.md`).
- State the trigger (when the rule applies) and the rule itself separately and explicitly.
- Explain *why* the constraint raises quality, with smells that indicate a violation.
- Record the model, tool, and version the rule targets.
