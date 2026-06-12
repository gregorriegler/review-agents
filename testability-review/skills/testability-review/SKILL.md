---
name: testability-review
description: Review a codebase for architectural testability — whether application logic can be exercised by fast, deterministic tests at a sound boundary. Orchestrates two stages: the style detector classifies each unit into a Style Classification Map, then the architecture reviewer applies the antipattern catalog filtered by that classification. Use when the user asks to review testability, assess the test boundary, check whether logic is reachable by fast tests, or find testability smells/antipatterns. Not for reviewing the quality of an existing test suite (a separate concern).
disable-model-invocation: true
---

# Testability Review

Orchestrates the two-stage testability feedback system over a target codebase and
presents a single consolidated report. Stage 1 classifies; stage 2 reviews against
that classification. The stages must run in order — the reviewer's findings are
meaningless without the classification, because the same construct is a violation
in one style and the intended design in another.

Both stages ship in this plugin as auto-registered subagents:
- `testability-style-detector` (stage 1) — emits the **SCM**
- `testability-architecture-reviewer` (stage 2) — consumes the SCM

The SCM (Style Classification Map) is the handoff contract between them, defined in
`style-classification-map.md` at the plugin root.

## Target

Determine what to review from the user's request:
- An explicit path or subdirectory → review that.
- "this repo" / no argument → the current project root.
Pass the same target to both stages so they classify and review the same code.

## Process

**1. Classify (stage 1).** Spawn the detector via the Agent tool
(`subagent_type: testability-style-detector`). Tell it the target and ask it to
produce a Style Classification Map. Capture its full output — that is the SCM.

**2. Surface the classification.** Before reviewing, show the user the SCM's repo
summary, especially **BBM units** (no discernible style) and **low-confidence /
internal-conflict** units. These are valuable on their own: for them the headline
recommendation is "choose a target style first," not "fix individual smells." If
the *entire* target is BBM or uniformly low-confidence, say so plainly and ask
whether to proceed to a provisional review or stop at the classification.

**3. Review (stage 2).** Spawn the reviewer
(`subagent_type: testability-architecture-reviewer`). Pass it **the full SCM
verbatim** plus the target. It applies the antipattern catalog filtered per unit
and returns findings (rule ID, severity, evidence, impact, minimal remediation).

**4. Present one report.** Combine into a single deliverable:
- the SCM summary (classification per unit + confidence),
- findings grouped by unit, ordered by severity then change-frequency/centrality,
- provisional findings clearly marked, with the assumed target style stated.

Do not re-run or second-guess the classification; if the reviewer surfaced a
contradiction with a label, present it as an open question, not a reclassification.

## Run the stages as subagents, not inline

Each stage is a focused agent with its own system prompt and tool set; running it
inline in the main thread loses that. Always spawn both via the Agent tool.
