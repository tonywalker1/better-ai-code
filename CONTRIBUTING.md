# Contributing to better-ai-code

Thanks for your interest in contributing. This project explores a single hypothesis: that AI models
already contain expert-level coding capability and can be reliably nudged to express it. Every
contribution — a prompt, a skill, a rule, or an experimental result — should serve that exploration.

By participating you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md).

## Ways to contribute

- **Artifacts** — new prompts, skills, or rules that push an AI toward exceptional output.
- **Experiments** — reproducible before/after comparisons showing an artifact's effect.
- **Evidence** — results that *support or refute* the hypothesis. Negative results are valued; an
  artifact that doesn't help is a finding, not a failure.
- **Documentation** — clearer explanations, examples, and structure.

## Ground rules

- **Show your work.** An artifact should come with the rationale for *why* it improves output and,
  where practical, a concrete before/after example.
- **One concern per change.** Keep prompts, skills, and rules focused; prefer several small,
  composable artifacts over one large one.
- **Reproducibility.** Document the model, tool, and version an experiment was run against, since
  results are model- and version-specific.
- **Tool-agnostic where possible.** Claude Code is the current focus, but separate the underlying
  idea from tool-specific mechanics so artifacts can be ported later.

## Workflow

1. **Open an issue first** for anything beyond a small fix, so the approach can be discussed before
   you invest effort.
2. **Fork and branch.** Use a descriptive branch name (e.g. `prompt/expert-error-handling`).
3. **Make your change.** Follow the conventions in existing artifacts and in this repo's style.
4. **Commit messages** use the imperative mood ("Add expert error-handling prompt", not "Added...").
5. **Open a pull request** describing the change, the reasoning, and any evidence of its effect.

## Code and content style

- Wrap Markdown at 120 characters.
- Indent with four spaces unless a format requires otherwise.
- Use absolute, unambiguous terminology and concrete examples over abstract description.

## Recognition

All contributors are added to [CONTRIBUTORS.md](CONTRIBUTORS.md). Please feel free to add yourself in
the same pull request as your first contribution.

## License

By contributing, you agree that your contributions will be licensed under the
[Apache License 2.0](LICENSE) that covers this project. Per Section 5 of the license, contributions
you intentionally submit for inclusion (e.g., via a pull request) are licensed under those terms
automatically, with no separate CLA required.
