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

- **A flaky test is a bug with a reproduction rate** — you raise the rate and find the race; you never retry it away (`diagnosing-flaky-tests`, `triage-flaky`).
- **A test owns its data** — most "flaky" suites are data-ownership violations wearing a timing costume (`test-data-management`).
- **A locator is a bet on what won't change** — bet on what the user sees, not on markup (`locator-strategy`).
- **E2E is for journeys; everything else goes down a layer** — variations and error contracts belong at the API layer at a fraction of the cost (`writing-e2e-tests`, `api-testing`).
- **A manual test case is not a spec** — extract the promise, then pick the cheapest layer that proves it (`automate-scenario`).
- **Tests can lie** — tautologies, vacuous assertions, and retry-as-fix make a suite worse than no suite (`test-review`, `audit-test-suite`).

## Reference

Skills split on one axis — who can invoke them. **User-invoked** skills are reachable only when you type them (e.g. `/audit-test-suite`); they orchestrate a session. **Model-invoked** skills fire automatically when the task fits; they hold the reusable discipline the orchestrators lean on.

### User-invoked

- **[setup-sdet-skills](./skills/automation/setup-sdet-skills/SKILL.md)** — Configure a repo for these skills: framework, layout, data strategy, locator convention, tags, retry policy → `docs/agents/testing.md`. Run once per repo.
- **[automate-scenario](./skills/automation/automate-scenario/SKILL.md)** — Turn a user story, manual test case, or bug report into automated tests: interview for the promise, split across layers, implement one verified slice at a time.
- **[audit-test-suite](./skills/automation/audit-test-suite/SKILL.md)** — Scan a whole suite for trust, stability, cost, and coverage findings; deliver a prioritized report with a verdict and a top three.
- **[triage-flaky](./skills/automation/triage-flaky/SKILL.md)** — Drain a flaky-test backlog: rank by signal damage, diagnose the worst, and leave every test fixed, quarantined with a ticket, or deleted.

### Model-invoked

- **[writing-e2e-tests](./skills/automation/writing-e2e-tests/SKILL.md)** — What a UI e2e test is (a user journey) and the laws that follow: independence, signal-based waiting, user-visible assertions. With an [anti-pattern catalogue](./skills/automation/writing-e2e-tests/anti-patterns.md).
- **[locator-strategy](./skills/automation/locator-strategy/SKILL.md)** — The selector ladder (role → label → text → test id → structure-free CSS) and what to do when a locator breaks.
- **[api-testing](./skills/automation/api-testing/SKILL.md)** — Contract-level assertions, deliberate unhappy paths, verifying through the interface, and pushing coverage below the UI.
- **[test-data-management](./skills/automation/test-data-management/SKILL.md)** — Factories with overrides, collision-proof identity, isolation, and cleanup that survives failure.
- **[diagnosing-flaky-tests](./skills/automation/diagnosing-flaky-tests/SKILL.md)** — The diagnosis loop: raise the reproduction rate, name the race, fix it, prove it with the same loop.
- **[test-review](./skills/automation/test-review/SKILL.md)** — Four review axes for test code: does it test behavior, can the assertion lie, will it hold up in the suite, does it read as a spec.

## Pairs well with

These skills focus on the test-automation domain. For the general engineering loop — grilling sessions, TDD, PRDs, triage — use [mattpocock/skills](https://github.com/mattpocock/skills) alongside them.

## License

MIT
