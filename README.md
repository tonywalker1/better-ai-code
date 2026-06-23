# better-ai-code

> Nudge AI to generate exceptional code all of the time.

## Hypothesis

The quality of code an AI model is trained on roughly follows a normal distribution. The model
therefore *sees* advanced, expert-level technique — but it rarely *produces* it unprompted, because
typical (median) code dominates the training signal. Left to its own devices, the model regresses
toward the mean and emits competent-but-ordinary code.

The bet of this project: **the expert capability is already in the model; we can reliably pull it to
the surface.** With the right prompts, skills, rules, and conventions, we can nudge — or push — an AI
to generate exceptional code *consistently*, instead of only when a request happens to land in the
upper tail of the distribution.

## What this repo is

A growing, version-controlled collection of the artifacts we develop while testing that hypothesis:

- **Prompts** — reusable instructions that bias generation toward expert technique.
- **Skills** — packaged, invocable procedures that encode a high-quality way of doing a task.
- **Rules** — path- or context-scoped constraints that enforce quality automatically.
- **History** — experiments, before/after comparisons, and notes on what actually moved the needle
  (and what didn't).

Everything here is treated as infrastructure-as-code for AI behavior: a single source of truth that
is reviewable, reproducible, and version-controlled.

## Scope

- **Now:** Claude Code. The first artifacts target Claude Code's prompt, skill, and rule mechanisms,
  because that is the tool in daily use.
- **Later:** other AI coding tools and models. The underlying ideas (surface latent expert
  capability; resist regression to the mean) are not Claude-specific, and the repo is structured to
  grow toward broader support.

## Status

Early and exploratory. Expect the structure, vocabulary, and conventions to evolve as experiments
teach us what works. Findings — positive and negative — are part of the deliverable.

## Contributing

Contributions, experiments, and counter-evidence are all welcome. See [CONTRIBUTING.md](CONTRIBUTING.md)
for guidelines and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for community expectations. Contributors
are recognized in [CONTRIBUTORS.md](CONTRIBUTORS.md).

## License

Apache License 2.0 — see [LICENSE](LICENSE) and [NOTICE](NOTICE).
