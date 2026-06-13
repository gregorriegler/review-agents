# Test Style Map (TSM)

The Test Style Map (TSM) is the handoff artifact of the test-review feedback
system. The test-style detector produces exactly one TSM for a codebase; the
smell reviewer consumes it. It is the single source of truth for "what is each
cluster of tests trying to be?" — so that a smell is read in context, because the
same construct is intended design in one test style and a defect in another.

The detector classifies; it does not judge. The TSM therefore carries
classifications and evidence, never smells or remediations. The reviewer attaches
its findings *to* clusters in the map; it does not re-derive it.

## The vocabulary (closed)

A cluster is classified on two axes plus three attributes. Each is a closed set.

### Axis 1 — Boundary (exactly one per cluster)

| ID | Style | Reaches |
|----|-------|---------|
| MT | Microtest | a single domain object, value object, or pure function; deep, below the app boundary |
| BU | Behavioral / sociable unit (aliases: acceptance, sociable unit) | the whole application logic; infrastructure excluded at the edges |
| CT | Component / UI | a component tree on a fast testbed; the network/HTTP boundary stubbed |
| IT | Integration (narrow) | one adapter against real or in-memory infrastructure |
| CDC | Contract | the agreement between two sides (consumer/provider) |
| E2E | End-to-end / system | the deployed stack through its real entry point |

### Axis 2 — Assertion strategy (one or more per cluster)

| ID | Strategy |
|----|----------|
| EXPECT | Example-based — arranged input → specific asserted value (the default) |
| VERIFY | Approval / snapshot — whole-output capture, verified file, nondeterminism scrubbed |
| PROPERTY | Property-based — generated inputs, asserted invariants, shrinking |

### Attribute — Double policy (exactly one)

`sociable` | `solitary` | `nullable` | `none`

How the cluster handles its dependencies. `none` = no doubles at all (typical of
MT). `sociable` = real collaborators, fakes only at infra ports (typical of BU).
`solitary` = collaborators replaced by mocks (London/mockist). `nullable` =
Shore-style: real SUT, infrastructure disconnected via `createNull()` embedded in
the production wrapper, side effects verified by output tracking.

### Attribute — Paradigm (exactly one)

`OO` | `functional` | `mixed`

Governs the *form* of every remediation: a page object vs a helper function, a
builder class vs a factory function. Recorded so the reviewer stays in-idiom.

### Attribute — Construction idiom (exactly one)

`object-mother` | `builder` | `factory-fn` | `fixtures` | `inline`

How the cluster builds its test data. `object-mother` — named canned instances.
`builder` — fluent, defaulted. `factory-fn` — plain factory with overrides.
`fixtures` — framework fixtures / shared arrange. `inline` — raw constructors in
the test body, no helper.

## Shape

A TSM has two parts: a **repo summary**, then one **cluster entry** per cluster.

### Repo summary

- **Classification unit** — what a "cluster" is here (a suite, a directory, a
  file group sharing one shape) and why that unit was chosen.
- **Overview table** — cluster → boundary → assertion → double policy → paradigm
  → construction → confidence.
- **Overall coherence** — `coherent` (one shape throughout), `mixed` (several
  coherent shapes), or `none`.
- **Flagged for human attention** — clusters with low confidence, internal
  conflict (the cluster argues with itself), or drift from declared intent.
- **SCM cross-reference** — if a Style Classification Map exists, note where a
  test style corroborates or contradicts the production style (e.g. production
  `NU` ⇒ tests expected `nullable`).

### Cluster entry

```
cluster:             <name> (<path or glob>)
boundary:            MT | BU | CT | IT | CDC | E2E
assertion_strategy:  EXPECT | VERIFY | PROPERTY        (one or more)
double_policy:       sociable | solitary | nullable | none
paradigm:            OO | functional | mixed
construction:        object-mother | builder | factory-fn | fixtures | inline
confidence:          high | medium | low
internal_conflict:   yes | no
evidence:
  - <file:loc> — <what this shows>
  - ...
drift:               <declared vs. actual, or "n/a">
open_questions:      <what to resolve before the reviewer fully trusts this label>
```

## Field semantics (the contract)

- Every field draws from its closed set above; `assertion_strategy` may list more
  than one, every other field is exactly one value.
- `confidence` — `high`: several mutually reinforcing signals. `medium`: signals
  lean one way but are sparse or partly contradicted. `low`: weak, conflicting, or
  too little test code — say what would resolve it.
- `internal_conflict` — `yes` when the cluster contradicts itself (e.g. half the
  tests null infrastructure, half mock it). Distinct from low confidence, which is
  *insufficient* evidence rather than *contradictory* evidence.
- `evidence` — mandatory. Every classification cites concrete `file:loc`
  references. A cluster entry with no evidence is not a classification; lower its
  confidence and state what is missing.
- `inline` construction and `mixed` paradigm are recorded **neutrally** — they are
  common and not in themselves smells. The reviewer decides whether their absence
  or inconsistency rises to a finding.

## How the reviewer reads it

- Use the classification as given; do not re-derive it. If test code contradicts a
  label, raise it under that cluster's `open_questions`, do not silently
  reclassify.
- The cluster's axes and attributes select which smells apply and *flip* how a
  shared smell is read — nulled infrastructure is intended under `nullable` and a
  defect under `none`; a whole-object snapshot is the goal under `VERIFY` and a smell
  under `EXPECT`.
- Treat a cluster as provisional when `confidence` is `low` or `internal_conflict`
  is `yes`. Provisional clusters still receive findings, but only the universal
  smells apply and findings are marked provisional with the assumed style stated.
- Every remediation is phrased in the cluster's `paradigm` + `construction` idiom.
