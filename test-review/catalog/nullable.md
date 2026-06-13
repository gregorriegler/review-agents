# Nullables (Shore-style sociable tests)

Loaded when `double_policy` = nullable. The cluster runs the **real** SUT with real
collaborators broadly, infrastructure disconnected via `createNull()` embedded in
the production wrapper, side effects verified by **output tracking**. This flips
several base smells and adds one of its own. `universal.md` still applies.

- **SMELL-mock-over-null** — A mock or fake built for a dependency that has, or
  should have, a `createNull()`. Null it instead; add the missing `createNull()`
  factory where it doesn't exist yet.

## Exemptions (guardrails — nullable tests look mockish but aren't)

A nullable cluster loads only this file plus `universal.md`; `doubles.md` does not
load. So apply these as guardrails against the general test-smell instincts you
carry, not as overrides of another catalog file:

- **Output tracking is not spying or over-specification** — asserting on a
  tracked-output array (`output.data`) is an observable-outcome assertion, the
  Nullables substitute for mock verification of side effects. Do not flag it as
  asserting on internals or as over-specified verification.
- **`createNull()` data is not a tautology** — real wrapper code runs between the
  null seam and the assertion, so the base `SMELL-tautology` (in `universal.md`)
  does not apply to nulled inputs.
- **Nulled infrastructure is intended** — do not report `SMELL-infrastructure`
  (the base smell in `universal.md`); the null lives in production code, and
  building a separate test-only fake would itself be the smell
  (`SMELL-mock-over-null`).
- **Broad real collaboration is the design** — do not flag the real SUT running
  many real collaborators as over-mocking or as needing too many mocks. There are
  no mocks here to over-use or count.
