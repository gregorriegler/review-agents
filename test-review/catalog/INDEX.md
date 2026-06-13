# Catalog loading index

The smell catalog is split into files. The reviewer holds none of them in its
standing prompt — for each cluster in the Test Style Map (TSM) it loads exactly
the files this index maps from that cluster's fields, then applies them. A typical
approval-tests cluster loads four files (`universal` + `construction` + `approval`
+ `doubles`); it never sees the React or nullable rules.

`${CLAUDE_PLUGIN_ROOT}/catalog/` holds these files.

| Load when… | File | Covers |
|------------|------|--------|
| always | `universal.md` | naming, AAA structure, flow control, value-object mocking, duplicate doubles, monkeypatch, base assertions (missing/tautology/vague/cherry-pick, anonymous-expected, boolean-equality-assert), base determinism (time/random, sleep, infrastructure), shared state, ignored, and test-data construction (off-idiom construction, magic-value vs. defaulting builders, irrelevant-detail, verbose, the in-idiom remediation rule) |
| `double_policy` ∈ {none, sociable, solitary} | `doubles.md` | over-mock, mock-over-fake, too-many-mocks, spy-on-internals, over-specified-verify, mock-driven tautology — read per policy |
| `double_policy` = nullable | `nullable.md` | mock-over-null; exemptions for output tracking, `createNull()` data, nulled infrastructure |
| `assertion_strategy` ∋ VERIFY | `approval.md` | the VERIFY-specific block + suppressions (missing-assert, vague-assert, cherry-pick) |
| `assertion_strategy` ∋ PROPERTY | `property.md` | exemptions for seeded generation, magic-value, invariant/range assertions; flow-control note |
| `boundary` ∈ {CT, E2E} | `ui-component.md` | framework-agnostic UI smells: mocking framework internals/hooks, structural queries, test-id-first, inspecting internals, shallow render, low-fidelity interaction, unawaited async; screen/page-object requirement; external-boundary-stub exemption |
| `boundary` ∈ {IT, E2E} | `integration.md` | infrastructure intended, sleep→bounded-polling, broader-assertion tolerance |
| `boundary` = CDC | `contract.md` | contract-derived stand-in and provider-verification exemptions |

Exactly one of `doubles.md` / `nullable.md` loads per cluster (by `double_policy`).
Files loaded later **override** `universal.md` where they state an exemption or
suppression — e.g. `approval.md` suppresses the base `missing-assert`,
`integration.md` exempts the base `infrastructure` smell. When two loaded files
disagree, the more specific (style/boundary) file wins over `universal.md`.

## Self-containment rule (for catalog authors)

`universal.md` always loads, but any other style/boundary file may *not* co-load.
So every style file must be **self-contained against `universal.md` + itself**: it
may reference and override smells defined in `universal.md`, but must never depend
on a *sibling* style file (`doubles.md`, `approval.md`, …) being loaded. State an
exemption that would otherwise live in a sibling as a self-explanatory guardrail
against the reviewer's general instincts, not as a reference to that sibling's key.
