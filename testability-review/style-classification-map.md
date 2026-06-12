# Style Classification Map (SCM)

The Style Classification Map (SCM) is the handoff artifact of the testability
feedback system. The style detector produces exactly one SCM for a codebase;
every downstream reviewer (testability-architecture review, and any future
reviewer that needs style context) consumes it. It is the single source of
truth for "what testability shape is each part of this codebase trying to be?"

The detector classifies; it does not judge. The SCM therefore carries
classifications and evidence, never smells or remediations. Reviewers attach
their findings *to* units in the map; they do not re-derive it.

## Shape

An SCM has two parts: a **repo summary**, then one **unit entry** per
classification unit.

### Repo summary

- **Classification unit** — deployable, bounded context, or top-level package —
  and why that unit was chosen.
- **Overview table** — unit → detected style(s) → confidence.
- **Overall strategy** — `coherent` (one strategy throughout), `mixed`
  (several coherent strategies), or `none`.
- **`BBM` units** — every unit with no discernible style, named and called out
  first and prominently. This is a top-level finding, not just a table row: the
  remediation downstream is "choose a target style first," not "fix smells."
- **Flagged for human attention** — units with low confidence, internal
  conflict, or declared-vs-actual drift.

### Unit entry

```
unit:            <name> (<path>)
declared_style:  <from ADRs/docs/strong conventions, or "none found">
detected_style:  <one or more of PA | FC | AF | ES | NU | FX | TS | BBM>,
                 or a list of candidate styles when signals genuinely conflict
confidence:      high | medium | low
internal_conflict: yes | no   # yes when the unit's own signals contradict each other
evidence:
  - <file:loc> — <what this shows>
  - ...
drift:           <declared vs. actual, or "n/a">
open_questions:  <what the team should resolve before a reviewer fully trusts this label>
```

## Field semantics (the contract)

- `detected_style` — the style vocabulary is closed: `PA`, `FC`, `AF`,
  `ES`, `NU`, `FX`, `TS`, `BBM`. `FC` (Functional Core / FCIS) and `AF`
  (A-Frame) are both logic-isolated-core styles — logic with zero infrastructure
  dependency. Combinations are normal and are expressed as a list (e.g.
  `AF + NU`). Candidate styles (unresolved
  ambiguity) are also a list, distinguished from a combination by `confidence`
  and `open_questions`.
- `confidence` — `high`: several mutually reinforcing signals, ideally
  declared intent agrees. `medium`: signals lean one way but are sparse or
  partly contradicted. `low`: weak, conflicting, or too little code — say what
  would resolve it.
- `internal_conflict` — `yes` when the unit argues with itself (e.g. half the
  modules invert dependencies, half don't). Distinct from low confidence, which
  is about *insufficient* evidence rather than *contradictory* evidence.
- `evidence` — mandatory. Every classification cites concrete `file:loc`
  references. A unit entry with no evidence is not a classification; lower its
  confidence and state what is missing.
- `BBM` (Big Ball of Mud) — reserved for genuine chaos: logic smeared
  across transport, data access, and framework callbacks with no organizing
  principle. A tidy, consistent per-request script style is `TS`, not `BBM`.

## How consumers read it

- Use the classification as given; do not re-derive it. If code contradicts
  a label, raise it under that unit's `open_questions` rather than silently
  reclassifying.
- Treat a unit as provisional when `detected_style` is `BBM`, `confidence`
  is `low`, or `internal_conflict` is `yes`. Provisional units still receive
  findings, but only the universal rules apply and findings are marked
  provisional with the assumed target style stated.
- The chosen `detected_style` selects which rules apply and what
  the correct remediation is — the same construct is a violation in one style
  and the intended design in another.
