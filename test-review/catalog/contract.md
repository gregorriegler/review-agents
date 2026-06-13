# Contract tests

Loaded when `boundary` = CDC. The test pins the agreement between two sides
(consumer/provider) — Pact, Spring Cloud Contract, or a hand-rolled contract
artifact. `universal.md` applies, with these exemptions.

## Exemptions (guardrails against general test-smell instincts)

- **Contract-derived stand-in is intended** — the other side is generated from /
  replayed against the contract, not a hand-written mock. Do not flag it as a mock
  that should be a fake, or as over-mocking; the contract is the mechanism. (If
  `doubles.md` also loaded for this cluster, this overrides its `mock-over-fake`.)
- **Provider verification is not a tautology** — verifying that the provider
  conforms to the contract (or that the consumer's expectations are met) is real
  behavior under test, not the base `SMELL-tautology` (in `universal.md`).

## Emphasized

- **Contract artifact present** — a CDC cluster should reference an actual shared
  contract. A "contract test" with no contract artifact, asserting against an
  inline hand-written model both sides could drift from, is the real defect — flag
  it as a missing/untrusted contract rather than exempting it.
