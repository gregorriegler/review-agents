---
name: test-review
description: Review a test suite for test smells, read in the context of what each cluster of tests is trying to be. Orchestrates two stages: the test-style detector classifies each cluster into a Test Style Map (boundary, assertion strategy, double policy, paradigm, construction idiom), then the smell reviewer applies a style-filtered smell catalog. Use when the user asks to review tests, find test smells, assess test quality, or check how a suite is written. Not for reviewing whether the architecture supports fast tests (that is testability-review, a separate concern).
disable-model-invocation: true
---

# Test Review

Stage 1 classifies; stage 2 reviews against that classification, and the result is
presented as a single consolidated report. The stages must run in order — the
reviewer's findings are meaningless without the classification, because the same
construct is a smell in one test style and the intended design in another (a nulled
database, a whole-output snapshot, a mocked collaborator).

Both stages ship in this plugin as auto-registered subagents:
- `test-style-detector` (stage 1) — emits the **TSM**
- `test-smell-reviewer` (stage 2) — consumes the TSM

The TSM (Test Style Map) is the handoff contract between them, defined in
`${CLAUDE_PLUGIN_ROOT}/test-style-map.md`.

## Relationship to testability-review

This reviews the **quality of the existing test suite** (smells, doubles, naming,
assertions). The separate `testability-review` plugin reviews whether the
**architecture supports fast tests**. They are complementary: if a Style
Classification Map from `testability-review` exists, pass it along so the detector
can corroborate test style against production style.

## Target

Determine what to review from the user's request:
- An explicit path or subdirectory → review that.
- "this repo" / no argument → the current project root.
Pass the same target to both stages so they classify and review the same tests.

## Process

**1. Classify (stage 1).** Spawn the detector via the Agent tool
(`subagent_type: test-style-detector`). Tell it the target and ask it to produce a
Test Style Map. Capture its full output — that is the TSM.

**2. Surface the classification.** Before reviewing, show the user the TSM's repo
summary, especially **low-confidence** and **internal-conflict** clusters. If the
suite's shape is unclear or contradictory, say so plainly and ask whether to
proceed to a provisional review or stop at the classification.

**3. Review (stage 2).** Spawn the reviewer (`subagent_type: test-smell-reviewer`).
Pass it **the full TSM verbatim** plus the target. It applies the style-matched
smell catalog per cluster and returns findings (smell key, evidence, minimal
in-idiom remediation).

**4. Present one report.** Combine into a single deliverable:
- the TSM summary (classification per cluster + confidence),
- findings grouped by file, each tagged with its cluster's style, ordered by
  severity then change-frequency/centrality,
- provisional findings clearly marked, with the assumed style stated.

Do not re-run or second-guess the classification; if the reviewer surfaced a
contradiction with a label, present it as an open question, not a reclassification.

Spawn both stages via the Agent tool, never inline: each is a focused agent with
its own system prompt and tool set, which running inline in the main thread loses.
