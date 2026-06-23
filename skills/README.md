# Skills

Packaged, invocable procedures that encode a high-quality way of doing a task.

Where a prompt nudges, a skill is a repeatable, named procedure: the steps, checks, and conventions
that produce expert output for a specific task, bundled so an AI can invoke it on demand.

## What belongs here

- Claude Code skills (the current focus) and, later, equivalents for other tools.
- Each skill in its own subdirectory, named for the task it performs.

## Conventions

- One skill per subdirectory (`skills/<skill-name>/`).
- Document the task it automates, when to invoke it, and the quality bar it enforces.
- Keep skills focused and composable — prefer several small skills over one large one.
- Record the model, tool, and version the skill targets.
