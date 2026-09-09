# SDET Skills

[![skills.sh](https://skills.sh/b/milosmilicevicsf/sdet-skills)](https://skills.sh/milosmilicevicsf/sdet-skills)

Agent skills for test automation engineers. Small, composable, tool-agnostic disciplines (with Playwright/TypeScript examples) that make coding agents write automation you can actually trust — not just tests that pass.

The format and philosophy follow [mattpocock/skills](https://github.com/mattpocock/skills): each skill is a small `SKILL.md` you can read in two minutes, adapt to your project, and compose with the others.

## Quickstart

1. Install with the [skills.sh](https://skills.sh) installer:

```bash
npx skills@latest add milosmilicevicsf/sdet-skills
```

2. Pick the skills you want and the agents to install them on. **Make sure you select `/setup-sdet-skills`.**

3. Run `/setup-sdet-skills` in your agent, in your test automation repo. It records your framework, directory layout, data strategy, locator convention, and tag/CI facts into `docs/agents/testing.md` — so every other skill reads your conventions instead of guessing them.

## Why These Skills Exist

Coding agents are good at producing tests and bad at producing *trustworthy* tests. Left alone, they transcribe manual steps into browser commands, sprinkle sleeps until it goes green, pin selectors to markup, and mark whatever still fails as `retries: 3`. Six months later you have a suite nobody believes.

These skills encode the disciplines that prevent that:

- **A risk found in refinement costs a sentence; found in production, an incident** — analyze the story before the code exists: blast radius, testability, unhappy paths (`analyze-story`).
- **The pyramid is a cost model, not a dogma** — every behavior at the lowest layer that proves it; an e2e must pay rent or move down (`test-pyramid`).
- **A flaky test is a bug with a reproduction rate** — you raise the rate and find the race; you never retry it away (`diagnosing-flaky-tests`, `triage-flaky`).
- **A fix changes the logic, not the odds** — sleeps, retry loops, soft assertions and force clicks raise the pass rate and leave the race in place (`fixing-flaky-tests`).
- **A test owns its data** — most "flaky" suites are data-ownership violations wearing a timing costume (`test-data-management`).
- **A dataset is code** — seeded, synthetic, rebuildable with one command; never a hand-curated database or a production copy (`generating-test-data`).
- **A locator is a bet on what won't change** — bet on what the user sees, not on markup (`locator-strategy`).
- **E2E is for journeys; everything else goes down a layer** — variations and error contracts belong at the API layer at a fraction of the cost (`writing-e2e-tests`, `api-testing`).
- **The cheapest layer that proves a behavior wins** — UI states and rendering logic belong in component tests, not another end-to-end (`component-testing`).
- **A bug report is a reproduction, not a complaint** — minimal steps, owned data, a named oracle, severity by impact (`reporting-bugs`).
- **A page object is an API, not a selector bag** — it exposes what a user can do and keeps the promise in the test, or it just relocates the duplication (`page-object-model`).
- **Mock the edge you don't own, never the thing under test** — a stub with no link to the real contract is a green light for a broken integration (`network-mocking`).
- **A manual test case is not a spec** — extract the promise, then pick the cheapest layer that proves it (`automate-scenario`).
- **Scripted tests prove the predicted; exploration hunts the rest** — charter-driven sessions that end in filed findings, never in "looks fine" (`explore-feature`).
- **Green CI is not release evidence** — ship on proof per change and named residual risks, not on the absence of red (`assess-release`).
- **Tests can lie** — tautologies, vacuous assertions, and retry-as-fix make a suite worse than no suite (`test-review`, `audit-test-suite`).

## The Flow

These skills chain the same way [mattpocock/skills](https://github.com/mattpocock/skills) chain (`/grill-with-docs` → `/to-spec` → `/to-tickets` → implement → `/code-review`) — and they're designed to plug into that flow, not replace it.

**The core loop** — new coverage for a feature, story, or bug report:

1. **`/setup-sdet-skills`** — once per repo. Records framework, layout, data strategy, locator convention, and retry policy into `docs/agents/testing.md`, so every later step reads facts instead of guessing.
2. **`/analyze-story`** — per story, during refinement, before any code exists. Maps the blast radius on the rest of the system, grills the story for checkability and unhappy paths, ranks the risks, and names the cheapest detection point for each — a refinement decision, a test at a layer (per `test-pyramid`), or an operational check.
3. **`/automate-scenario`** — per scenario, once the story is implemented. Starts from the `/analyze-story` analysis when one exists; otherwise the grilling is built in: an interview extracts the *promise* (capabilities, unhappy paths), the *layer split* (what needs a browser vs. what the API layer proves cheaper), and the *oracle* (where expected values come from). Then implementation runs one verified vertical slice at a time — and the discipline skills (`writing-e2e-tests`, `component-testing`, `locator-strategy`, `page-object-model`, `api-testing`, `network-mocking`, `test-data-management`) fire automatically as the code gets written.
4. **`/explore-feature`** — per feature, once it's testable. A charter-driven exploratory session (charters come straight from the `/analyze-story` risk register) that hunts what no script predicted, files findings per `reporting-bugs`, and hands candidate regressions back to `/automate-scenario`.
5. **`test-review`** — fires on test code in review: would this test catch its bug, and can it lie?

**The maintenance loop** — keeping an existing suite trustworthy:

- **`/audit-test-suite`** every few weeks — a whole-suite scan for trust, stability, cost, and coverage findings, prioritized.
- **`/triage-flaky`** when the flaky backlog grows — rank by signal damage, diagnose the worst via `diagnosing-flaky-tests`, repair them to the `fixing-flaky-tests` bar, and leave every test fixed, quarantined with a ticket, or deleted.
- **`/assess-release`** before each ship — what changed, what proves each change, what debt rides along; a go/no-go where every residual risk is named with its detection in production.

**Running both skill sets?** The seams: use Matt's `/grill-with-docs` → `/to-spec` → `/to-tickets` to spec the feature — `/analyze-story` slots into that same phase from the other side: his grilling designs the feature, this one asks what it breaks and where to prove it. When an issue is about *product code*, implement it with his `tdd`; when it's about *coverage*, run `/automate-scenario`. His `/code-review` and this repo's `test-review` are complementary axes on the same PR, and `/audit-test-suite` is to your test suite what his `/improve-codebase-architecture` is to your source.

## Reference

Skills split on one axis — who can invoke them. **User-invoked** skills are reachable only when you type them (e.g. `/audit-test-suite`); they orchestrate a session. **Model-invoked** skills fire automatically when the task fits; they hold the reusable discipline the orchestrators lean on.

### User-invoked

- **[setup-sdet-skills](./skills/automation/setup-sdet-skills/SKILL.md)** — Configure a repo for these skills: framework, layout, data strategy, locator convention, tags, retry policy → `docs/agents/testing.md`. Run once per repo.
- **[analyze-story](./skills/automation/analyze-story/SKILL.md)** — Analyze a story before implementation: blast radius on the rest of the system, testability grilling, a ranked risk register, and the cheapest detection point per risk.
- **[automate-scenario](./skills/automation/automate-scenario/SKILL.md)** — Turn a user story, manual test case, or bug report into automated tests: interview for the promise, split across layers, implement one verified slice at a time.
- **[explore-feature](./skills/automation/explore-feature/SKILL.md)** — A charter-driven exploratory session: attack heuristics, follow-the-smoke deviation, and a close that files bugs and names candidate regressions.
- **[assess-release](./skills/automation/assess-release/SKILL.md)** — Evidence-based go/no-go for a release candidate: change list by risk, proof per change, debt that rides along, residual risks named.
- **[audit-test-suite](./skills/automation/audit-test-suite/SKILL.md)** — Scan a whole suite for trust, stability, cost, and coverage findings; deliver a visual HTML report with a verdict and a top three.
- **[triage-flaky](./skills/automation/triage-flaky/SKILL.md)** — Drain a flaky-test backlog: rank by signal damage, diagnose the worst, and leave every test fixed, quarantined with a ticket, or deleted.

### Model-invoked

- **[test-pyramid](./skills/automation/test-pyramid/SKILL.md)** — The cost model behind the layers: what each layer alone proves, where e2e cost really lives, quantity as a symptom, and the shift-left detection ladder.
- **[writing-e2e-tests](./skills/automation/writing-e2e-tests/SKILL.md)** — What a UI e2e test is (a user journey) and the laws that follow: independence, signal-based waiting, user-visible assertions. With an [anti-pattern catalogue](./skills/automation/writing-e2e-tests/anti-patterns.md).
- **[component-testing](./skills/automation/component-testing/SKILL.md)** — The layer between unit and e2e: real component, real DOM, real events, data stubbed at the edge — where UI states and logic belong instead of another journey.
- **[locator-strategy](./skills/automation/locator-strategy/SKILL.md)** — The selector ladder (role → label → text → test id → structure-free CSS) and what to do when a locator breaks.
- **[page-object-model](./skills/automation/page-object-model/SKILL.md)** — Page objects as a behavior API, not a selector bag: capabilities over getters, assertions in the test, and how to audit a layer that hurts.
- **[api-testing](./skills/automation/api-testing/SKILL.md)** — Contract-level assertions, deliberate unhappy paths, verifying through the interface, and pushing coverage below the UI.
- **[network-mocking](./skills/automation/network-mocking/SKILL.md)** — Mock the boundary you don't own, never the thing under test; keep stubs tied to the real contract and assert the outcome, not the call.
- **[test-data-management](./skills/automation/test-data-management/SKILL.md)** — Factories with overrides, collision-proof identity, isolation, and cleanup that survives failure.
- **[generating-test-data](./skills/automation/generating-test-data/SKILL.md)** — Data at dataset scale: idempotent seeds as repo code, deliberately ugly and skewed synthetic data, seeded randomness, synthesis over production copies.
- **[diagnosing-flaky-tests](./skills/automation/diagnosing-flaky-tests/SKILL.md)** — The diagnosis loop: raise the reproduction rate, name the race, fix it, prove it with the same loop.
- **[fixing-flaky-tests](./skills/automation/fixing-flaky-tests/SKILL.md)** — The bar the repair must clear: replace the assumption with a guarantee, in the smallest diff that holds — with the counterfeit-fix catalogue (retries, sleeps, poll loops, soft asserts, conditional flow, force clicks) and the two proofs.
- **[test-review](./skills/automation/test-review/SKILL.md)** — Four review axes for test code: does it test behavior, can the assertion lie, will it hold up in the suite, does it read as a spec.
- **[reporting-bugs](./skills/automation/reporting-bugs/SKILL.md)** — Turn a found defect into a report a developer can act on: minimal reproduction, named oracle, one defect per report, severity by impact.

## Pairs well with

These skills focus on the test-automation domain. For the general engineering loop — grilling sessions, TDD, PRDs, triage — use [mattpocock/skills](https://github.com/mattpocock/skills) alongside them.

## License

MIT
