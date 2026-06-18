# review-agents

A Claude Code plugin marketplace of code-review agents. Each plugin ships a
focused reviewer that inspects a diff or codebase along one dimension and
reports findings with evidence and minimal remediations.

## Plugins

| Plugin | Type | What it reviews |
| --- | --- | --- |
| `testability-review` | Two-stage | Classifies each unit into a Style Classification Map, then applies a style-filtered antipattern catalog to flag architectural-testability issues. |
| `test-review` | Two-stage | Classifies each test cluster into a Test Style Map (boundary, assertion strategy, double policy, paradigm, construction idiom), then applies a style-filtered smell catalog. |
| `modularity-review` | Single-stage | Scores coupling along two axes — Strength (how much pieces know about each other) and Distance (how far apart they sit) — and flags spots where both are high. |
| `abstraction-review` | Single-stage | Examines boundaries meant to hide a decision (interfaces, wrappers, service layers, adapters, facades) and flags pass-through, leaky, sprawling, or ceremonial abstractions. |

## Install

Add this repository as a marketplace, then install the plugins you want:

```
/plugin marketplace add gregorriegler/review-agents
/plugin install testability-review
/plugin install test-review
/plugin install modularity-review
/plugin install abstraction-review
```

## Use

- `test-review` and `testability-review` expose slash commands: `/test-review`
  and `/testability-review`.
- `modularity-review` and `abstraction-review` ship subagents: ask to review
  modularity or abstractions to spawn them.

## Layout

```
.claude-plugin/marketplace.json   marketplace manifest
<plugin>/.claude-plugin/plugin.json   plugin manifest
<plugin>/agents/                  reviewer subagents
<plugin>/skills/                  slash-command skills
<plugin>/catalog/                 smell / antipattern catalogs
```

## License

MIT. See [LICENSE](LICENSE).
