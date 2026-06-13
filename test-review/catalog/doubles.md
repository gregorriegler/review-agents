# Mocks & test doubles

Loaded when `double_policy` ∈ {none, sociable, solitary}. (For `nullable`, load
`nullable.md` instead.) Read each smell against the policy.

These three policies share this file but are **not** interchangeable:
`SMELL-over-mock` reads each one differently — the same double is always-wrong
under `none`, wrong-only-inside-the-logic under `sociable`, and intended design
under `solitary`. Read your cluster's value before applying any smell here.

- **SMELL-over-mock** — A double for a collaborator the style says should be real.
  - `none` (typical MT) — **any** double at all is a smell; collaborators run real all the way down.
  - `sociable` (typical BU) — a double for an in-memory collaborator is a smell; fakes are allowed **only** at infra ports. A double *inside* the logic is the finding.
  - `solitary` (London/mockist) — mocking collaborators is the **intended design**; do not report it.

  Resolving port vs. logic under `sociable` (this is the whole judgment): if an
  SCM is available, treat the types it classifies as adapters/ports as the infra
  ports — a double for one of those is allowed, a double for anything it classifies
  as application/domain logic is the finding.

  With no SCM, **open the doubled type's implementation before flagging** — judge
  from the source, never from the name or the call site (`PricingService` is a
  thin HTTP client in one repo and a rules engine in another). Answer two questions
  against the implementation:
  - **Does it cross a process or I/O boundary?** It imports or calls a DB, network,
    filesystem, clock, or external service (repo, gateway, queue). ⇒ **port** —
    a double is allowed, no finding.
  - **Does it have in-process domain behavior worth asserting in its own right,
    with no I/O seam?** Computation, rules, branching over domain state. ⇒ **logic**
    — doubling it is `over-mock`, the finding.

  When the implementation genuinely does both, or you cannot locate it, treat the
  finding as **provisional**: emit it, but state the basis and that no SCM
  confirmed it (see the reviewer's reporting rule on stating the port/logic basis).

- **SMELL-mock-over-fake** — Per-test mock setup where a reusable fake would be
  better. *(Under CDC the contract-derived stand-in is exempt — see `contract.md`.)*

- **SMELL-too-many-mocks** — 3+ mocks to instantiate the SUT, signalling too many
  collaborators — consider splitting responsibilities in production code.
  - Live under `solitary`.
  - Inert under `none` (no mocks to count; a double there is already `over-mock`).

- **SMELL-spy-on-internals** — Asserting on internal calls, private state, or
  execution order instead of observable outcomes.

- **SMELL-over-specified-verify** — Exact call counts (`times(1)`), ordering
  (`InOrder`), `verifyNoMoreInteractions`. A behavior-preserving refactor should not
  break the test. Plain `verify(mock).method()` for a genuine side effect is fine.
  Most relevant under `solitary`.

- **SMELL-tautology (mock-driven)** — A mock is told to return X, then the test
  asserts X with no real production code in between. Litmus: would it still pass
  with all production code deleted? Complements the base `SMELL-tautology` in
  `universal.md`.
