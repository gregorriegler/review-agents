# Property-based tests

Loaded when `assertion_strategy` ∋ PROPERTY. Generated inputs, asserted invariants, a
shrinking framework. This exempts a few base smells; `universal.md` otherwise applies.

## Exemptions (do not report against a PROPERTY cluster)

- **Seeded generation is not `SMELL-time-random`** — framework-driven input
  generation that is reproducible and shrinkable is the design, not nondeterminism.
- **No fixed value to name `IRRELEVANT`** — `SMELL-magic-value` does not apply to
  generated inputs.
- **Invariant/range assertions are not `SMELL-vague-assert`** — a property
  legitimately asserts a relationship or bound over many inputs rather than one
  exact value. Still flag a genuinely weak property (e.g. asserting only that the
  result is non-null).
- **An invariant has no fixed `expected` to name** — `SMELL-anonymous-expected`
  does not apply to a property that asserts a relationship between generated input
  and output rather than a single named expected value.

## Still applies

- **SMELL-flow-control** — the *framework* owns iteration and shrinking; the test
  body (the property function) must still contain no hand-rolled `if`/`for`/`while`.
  Conditioning inputs belongs in generators/preconditions, not branches in the body.
