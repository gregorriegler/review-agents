# Integration & end-to-end tests

Loaded when `boundary` ∈ {IT, E2E}. IT exercises one adapter against real or
in-memory infrastructure; E2E drives the deployed stack. `universal.md` applies,
with these relaxations.

## Exemptions / relaxations (override `universal.md`)

- **Real infrastructure is intended** — do not report `SMELL-infrastructure`. A real
  repository over an in-memory DB (IT), or the real stack (E2E), is the point;
  mocking it would be the smell.
- **`SMELL-sleep` → bounded polling** — some real waiting is unavoidable here. Still
  a finding, but the remediation is bounded polling with a timeout, not removal of
  the wait. A fixed unconditional `sleep(n)` remains a smell.
- **Broader assertions tolerated for E2E smoke** — `SMELL-vague-assert` relaxes for
  a thin E2E smoke layer asserting reachability/health. Narrow IT round-trip tests
  should still assert the exact persisted/returned value.

## Still emphasized

- **SMELL-mega-test** — broad tests tempt many unrelated assertions in one flow.
  One behavior per test still holds; failure should pinpoint the cause.
