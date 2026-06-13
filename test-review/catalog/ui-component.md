# Component / UI tests

Loaded when `boundary` ∈ {CT, E2E}. CT renders a component subtree on a fast
testbed; E2E drives the real UI. The principles are framework-agnostic — they hold
for web (React Testing Library, Vue Test Utils, Testing Library family), mobile
(Espresso, XCUITest, Flutter widget tests), and desktop (WinAppDriver, Playwright
for Electron). Examples below are illustrative, not the rule. `universal.md`
applies; this adds the UI-specific smells and one exemption.

Guiding principle: **render the real thing and drive it the way a user does.** The
more a UI test resembles how the software is actually used, the more confidence it
gives. Substitute only the *external boundary*, never the component's own innards.

- **SMELL-mock-framework-internals** — Mocking the framework's own building blocks
  instead of rendering them for real: hooks / custom hooks (`useQuery`, a custom
  `useCart`), component state or lifecycle, the router, or the child components of
  the unit under test. The test then verifies the mock, not the component.
  Remediation: render the real component with its real hooks and children, and
  control the *external* boundary instead (stub the network/data source, the
  navigation target, the platform service). A genuinely heavy child (a map, a
  charting engine) may be substituted — but at *its* external boundary, not by
  replacing the child wholesale.

- **SMELL-structural-query** — Selecting elements by structure or position —
  CSS selectors, class names, DOM/view-tree traversal, child indexes
  (`container.querySelector('.btn')`, `findViewById`, `nth-child`) — instead of the
  semantic/accessible handle a user perceives (role, label, text, accessibility id,
  content description). Remediation: query by semantic handle (`getByRole`,
  `onView(withText(...))`, accessibility identifier). Where a third-party component
  exposes no semantic handle, isolate the structural selector in a **screen/page
  object** (`OO`) or **helper function** (`functional`) — `expandRow(grid, 3)`,
  never an inline structural selector in the test body.

- **SMELL-testid-first** — Reaching for a test-only id (`data-testid`, test tags) as
  the *first* choice when a semantic handle exists. Test-ids are an escape hatch for
  when nothing else works, not the default locator. Remediation: prefer the semantic
  handle; keep test-ids for the genuinely opaque cases.

- **SMELL-inspect-internals** — Asserting on component internals: local state,
  props, instance methods, or emitted-but-not-rendered values
  (`wrapper.state()`, `instance().handleClick()`) — the UI form of asserting on
  internals rather than observable behavior. Remediation: assert on what the user observes — rendered
  output, accessible state, navigation, and the calls that crossed the external
  boundary.

- **SMELL-shallow-render** — Shallow rendering (or otherwise stubbing children) to
  inspect structure rather than behavior. It couples the test to the component tree
  and tests nothing a user experiences. Remediation: render the full subtree and
  assert on behavior.

- **SMELL-low-fidelity-interaction** — Driving the UI by calling handler props
  directly (`props.onClick()`), dispatching synthetic low-level events, or otherwise
  bypassing real interaction, instead of simulating a real user (high-fidelity user
  interaction that carries focus, key, and pointer semantics — e.g. `userEvent` over
  `fireEvent`). Remediation: interact through the user-interaction API.

- **SMELL-unawaited-async** — Asserting on async UI synchronously, or waiting with a
  fixed sleep/timeout, instead of awaiting the framework's settle primitive
  (`findBy*`/`waitFor`, idling resources, `waitForIdle`). The leading cause of flaky
  UI suites. Remediation: await the framework's "settled" signal; never a fixed delay.

## Exemption

- **Stubbing the external boundary is intended (CT)** — do not report
  `SMELL-infrastructure` for stubbing the network/data/platform boundary in a
  component test; rendering real components against a stubbed boundary is the point
  of the fast testbed. (E2E drives the real stack and is also exempt.) This exemption
  is for the *external* boundary only — it never licenses
  `SMELL-mock-framework-internals`.
