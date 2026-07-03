---
name: audit-test-suite
description: Scan an entire test suite for trust, cost, and coverage problems, and present a prioritized findings report.
disable-model-invocation: true
---

# Audit Test Suite

Answer the question every team eventually asks: **can we trust this suite?** Scan the whole test codebase — not a diff — for the patterns that make suites lie, burn time, or miss what matters, then present findings the team can act on in priority order.

Read `docs/agents/testing.md` (if it exists) first; audit against the repo's declared conventions, not just general principles.

## 1. Inventory

Establish the shape of the suite before judging it: test counts per layer (unit / API / e2e), directories, tags, runtime if CI history is available, and the retry/quarantine configuration. The inventory itself often contains the headline finding — an inverted pyramid (300 e2e, 12 API tests), or `retries: 3` applied suite-wide.

## 2. Scan

Sweep the suite for each finding class. Use search (grep patterns, then read hits in context) so the pass is exhaustive, not anecdotal — every test file gets covered, and say so in the report.

- **Trust findings** — tests that can lie: tautological assertions, vacuous/conditional assertions, retry-as-fix, `skip`/`only` left enabled, tests asserting nothing beyond "no error". Judged per the `test-review` skill's axes.
- **Stability findings** — races waiting to fire: sleeps, conditional flow, shared accounts, hard-coded unique values, cleanup that skips on failure. Per the `writing-e2e-tests` anti-pattern catalogue and the `test-data-management` skill.
- **Cost findings** — spend without return: e2e tests provable at the API layer, mega-tests, duplicated journeys, structure-pinned locators (per the `locator-strategy` ladder), UI-driven setup.
- **Coverage findings** — the gaps: promises in the spec/PRD with no test at any layer, untested error contracts, journeys with assertions only at the end. Coverage here means *behavior* coverage — line-coverage numbers don't answer it.

For CI-history questions (which tests actually flake, what actually times out), use real run data if accessible; otherwise mark those findings as static-analysis-only.

## 3. Report

Deliver a prioritized report, not a lint dump:

1. **Verdict** — one paragraph: can this suite be trusted to gate releases, and what's the biggest reason it can/can't.
2. **Findings by severity** — trust, then stability, then cost, then coverage. Each finding: the pattern, file/line instances (grouped — one entry per pattern, not per occurrence), and the direction of the fix. Cap at what's actionable; 400 instances of one pattern is one finding with a count.
3. **The top three** — if the team fixes only three things, which, and why those.

Offer to turn accepted findings into tracked issues, and to fix the mechanical classes (locator climbs, sleep-to-signal conversions) directly — one pattern class per PR, so each is reviewable.
