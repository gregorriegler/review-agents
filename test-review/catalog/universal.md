# Universal smells

Always loaded. These apply to every cluster regardless of style. Some carry a base
reading here that a later-loaded style file may exempt or suppress (noted inline);
when that happens, the style file wins.

## Naming

- **SMELL-should** — Test name contains "should". Names state facts, not wishes.
- **SMELL-its-no-sentence** — `it('…')` doesn't read as a sentence starting with
  "it". `it('getMaxLevel returns max')` → `it('returns the maximum upgrade level')`.
- **SMELL-tech-naming** — Technical jargon instead of domain language.

## Structure

- **SMELL-aaa-comments** — Comments labeling AAA / Given-When-Then phases. The
  blank-line structure speaks for itself.
- **SMELL-aaa-spacing** — Missing blank-line separators between Arrange/Act/Assert.
  *(Relaxed under VERIFY — see `approval.md`.)*
- **SMELL-flow-control** — `if`, `else`, ternary, `for`, `while`, `do`,
  `try/catch/except` in a test body. Use parameterized tests for iteration.
  *(PROPERTY refines this — see `property.md`.)*
- **SMELL-nesting** — Excessive nesting or indentation that harms readability.
- **SMELL-mega-test** — Multiple unrelated behaviors asserted in one test. Split.
- **SMELL-illegible-intent** — The capstone, whole-test smell: a reader who has
  not seen the production code cannot tell *what behavior is protected or why* from
  the test alone. The cause-and-effect link between the `when` and the `then` isn't
  legible — the asserted outcome doesn't visibly follow from the exercised input.
  Unlike the local smells, every line may be individually fine (good name, clean
  AAA, no magic values) yet the test still reads as a sequence of operations rather
  than a statement of behavior. **Default to NOT flagging — this is a high bar.**
  Flag only when *both* hold: (1) intent genuinely cannot be reconstructed from the
  test by itself, and (2) no local smell already explains why — if `SMELL-its-no-sentence`,
  `SMELL-anonymous-expected`, `SMELL-irrelevant-detail`, `SMELL-mega-test`, or a
  construction smell already covers the cause, report *that* and stop; do not stack
  this on top. Report at most once per test, naming the specific question the reader
  is left with ("can't tell why this input yields this status"), not as a generic
  "hard to read." Remediation: make the behavioral link explicit — a name that
  states the rule, an `expected` tied to the input, or surfacing the one input that
  drives the outcome — never "add comments."

## Test doubles (data)

- **SMELL-mock-data** — Mocking a value object or data structure. Never.
- **SMELL-duplicate-double** — A new fake/mock/stub that duplicates an existing
  reusable double or helper. Reuse the existing one.
- **SMELL-monkeypatch** — Monkeypatching where dependency injection was possible.

## Assertions (base)

- **SMELL-missing-assert** — No assertion or verification. *(Suppressed under VERIFY —
  see `approval.md`.)*
- **SMELL-tautology** — Outcome predetermined by setup, independent of production
  code (would it still pass with all production code deleted?). Forms:
  `assertTrue(true)`, `assertNotNull(new Object())`, verifying framework behavior.
  *(Mock-driven form in `doubles.md`; exemptions in `nullable.md` / `contract.md`.)*
- **SMELL-vague-assert** — Accepts a broad range instead of the exact outcome
  (`Is.GreaterThanOrEqualTo(400)` → pin the status code). *(Relaxed under PROPERTY and
  E2E — see `property.md` / `integration.md`.)*
- **SMELL-cherry-pick** — Cherry-picked field assertions when the whole object
  should be verified. Build a complete `expected` with non-deterministic fields
  echoed from `actual`, compare in one assertion. *(Under VERIFY → `SMELL-ap-partial`;
  see `approval.md`.)*
- **SMELL-anonymous-expected** — An expected value that the reader must
  reverse-engineer: a literal whose *meaning* (not just its number) is unclear
  because it is computed, derived, or domain-loaded, and the test name does not
  already convey it. Name it (`expected`, or a domain term — `fullPriceWithTax`,
  `maxUpgradeLevel`). **Default to NOT flagging.** A self-evident literal tied to
  the test name (`it('returns 0 for an empty cart')` → `toBe(0)`) needs no name;
  only flag when naming genuinely removes a "why this value?" question — e.g. a
  magic constant like `toBe(4200)` with no nearby arithmetic showing where it came
  from. When the same unexplained constant recurs across a cluster, report it once
  as drift, not per occurrence. *(Exempt under PROPERTY — see `property.md`.
  Suppressed under VERIFY — see `approval.md`.)*
- **SMELL-boolean-equality-assert** — An equality or comparison expressed through
  `assertTrue`/a raw boolean (`assertTrue(a.equals(b))`, `assert x == 5`) instead
  of an equality assertion, discarding the `expected` vs `actual` framing and the
  diff on failure. Use the framework's equality assertion. *(Suppressed under
  VERIFY — see `approval.md`.)*

## Isolation & determinism (base)

- **SMELL-shared-state** — Shared mutable state between tests (class/module-level
  variables mutated without reset). Framework `@BeforeEach` re-init is fine.
- **SMELL-time-random** — Wall-clock time or random without a seam. Inject a clock
  or seed. *(Exempt under PROPERTY — see `property.md`.)*
- **SMELL-sleep** — `sleep`/`delay`/`Thread.sleep`. *(IT/E2E refine the fix — see
  `integration.md`.)*
- **SMELL-infrastructure** — Real network or filesystem where a fake would do; a
  finding only when it contradicts the cluster's declared boundary.
  *(IT/E2E, CT, and nullable clusters override — see the loaded style file.)*
- **SMELL-ignored** — `@Ignore`/`@Skip`/`@Disabled`. Fix it or delete it.

## Test-data construction

Every cluster has a `paradigm` and a `construction` idiom; these read against it.
`inline` construction and `mixed` paradigm are not in themselves smells — they
become findings only when duplication or verbosity actually appears.

- **SMELL-off-idiom-construction** — Test data built `inline`, or a new
  builder/mother/factory reinvented, when the cluster has an established idiom.
  Remediation in the cluster's `paradigm`: a builder/object-mother class for `OO`,
  a factory function for `functional`. When the cluster is `inline`-everywhere with
  no idiom yet, the remediation is "introduce the idiom that fits the paradigm,"
  not a blanket flag.
- **SMELL-magic-value** — A value that exists only to satisfy a signature is not
  marked as irrelevant. In an `inline` cluster, name it `IRRELEVANT` (or the
  language equivalent). **Flip:** under a defaulting `builder`/`factory-fn`/mother,
  an *unset* field already signals irrelevance — do not demand explicit
  `IRRELEVANT` markers on top of defaulting. *(Exempt under PROPERTY — see `property.md`.)*
- **SMELL-irrelevant-detail** — The same irrelevance principle beyond constructor
  values: arrange/setup, collaborators, or steps that do not drive the asserted
  outcome, and variable names drawn from implementation detail rather than the
  role the value plays in the scenario. Anything the reader does not need to
  understand the behavior is noise — remove it, push it behind a defaulting
  builder/mother/factory, or rename it to its role. Distinct from `SMELL-verbose`
  (ceremony in *how* data is built) and `SMELL-magic-value` (a single unmarked
  value); this is irrelevant *content and naming* across the test body.
- **SMELL-verbose** — Unnecessary ceremony in construction or setup. Shorter is
  better, but the fix is the idiom, not fewer lines: route construction through the
  builder/mother/factory so the arrange reads like a specification.

### Remediation form (applies to every finding, in any file)

Phrase every construction-, double-, and UI-helper remediation in the cluster's
`paradigm` + `construction` idiom: a **page object** vs a **helper function**, a
**builder class** vs a **factory function**. Never tell a functions-only suite to
"extract a class." Point at an in-repo example of the established idiom.
