# Approval / snapshot tests

Loaded when `assertion_strategy` ∋ VERIFY. A VERIFY test has a distinct shape, and the
example-based assertion smells give way to these. A good approval test builds its
fixture readably — ideally through a builder or object mother that reads as a
domain narrative — then calls a **single domain-named `verify*` wrapper** that
hides the approval call, the printed representation, and the scrubbing:

```
agent = createAgent()
            .llmAnswers("Foo")
            ...
verifyAgent(agent)
```

- **SMELL-ap-inline-approval** — the raw approvals/snapshot call (and its scrubbing
  or formatting) sits inline in the test body instead of behind one domain-named
  verify function. Remediation: extract `verify<Subject>(...)` that owns the
  approval call, the rendered representation, and the scrubbers; the test ends in a
  single readable verify line.
- **SMELL-ap-unreadable-fixture** — the fixture is assembled inline or verbosely
  instead of through the cluster's `construction` idiom. The arrange should read
  like a specification. Remediation: build it through a builder/object mother (`OO`)
  or factory function (`functional`).
- **SMELL-ap-unscrubbed** — nondeterministic fields (ids, timestamps, ordering,
  absolute paths) reach the approved output without scrubbing, making the test
  flaky or perpetually re-approved. Remediation: scrub inside the verify wrapper.
- **SMELL-ap-partial** — a hand-picked subset is asserted instead of approving the
  whole rendered outcome (the VERIFY form of `SMELL-cherry-pick`). Remediation: approve
  the whole readable representation.
- **SMELL-ap-opaque-output** — the approved artifact is an unreadable blob (minified
  JSON, a raw object dump) a human can't review in a diff. The value of approval is
  a *readable* verified artifact. Remediation: render a stable, human-readable
  printout in the verify wrapper.
- **SMELL-ap-multi-verify** — several unrelated approvals in one test, fragmenting
  the verified artifact. One verified outcome per behavior (combination approvals of
  related facets are fine).
- **SMELL-ap-received-committed** — an unapproved `.received`/`.snap.new` file is
  committed, or the `.approved`/`.verified` file is missing from version control.

## Suppressed / relaxed (override `universal.md`)

- **Suppressed:** `SMELL-missing-assert` (the verify *is* the assertion),
  `SMELL-vague-assert` (no value assertion to be vague), `SMELL-cherry-pick`
  (reported as `SMELL-ap-partial` instead), `SMELL-anonymous-expected` and
  `SMELL-boolean-equality-assert` (there is no value-level `expected`/`actual` to
  name or compare — the verified artifact is the expectation).
- **Relaxed:** `SMELL-aaa-spacing` — a VERIFY test is typically Arrange + a single
  verify line with no distinct Act/Assert blocks; flag missing separation only when
  the arrange itself has clearly separable phases.
