---
name: automate-scenario
description: Turn a user story, manual test case, or bug report into automated tests — interview first, then implement one slice at a time.
disable-model-invocation: true
---

# Automate Scenario

Take a scenario — user story, manual test case, acceptance criteria, bug report — and turn it into automated tests worth keeping. The failure mode this skill prevents: transcribing manual steps into browser commands and calling it automation. A manual test case describes *how a human checks*; an automated test asserts *what the product promises*. Extract the promise, then choose the cheapest layer that proves it.

Read `docs/agents/testing.md` (if it exists) before starting; it answers framework, layout, data, and tag questions so you don't re-ask them.

## 1. Interview

Interview the user **one question at a time**; if the codebase can answer a question, explore instead of asking. Resolve, in order:

1. **The promise.** What behavior must hold? Restate the scenario as capabilities ("user can X", "given Y the system Z") and confirm the list is complete — including the unhappy paths the manual case implies but doesn't spell out.
2. **The layer split.** For each capability: does proving it need a browser? Propose the split explicitly — e.g. "validation variants at the API layer, one happy-path journey e2e" — and get agreement. This is the decision that determines the suite's cost forever; per the `api-testing` skill, most variations belong below the UI.
3. **The oracle.** For each capability, where does the expected value come from — spec, existing behavior, the user's head? If the answer is "whatever the app currently does", flag it: that's a characterization test, worth having but worth labeling.
4. **Reality check.** Does the feature exist and work today? Automating against a broken feature produces a red suite nobody trusts; note known bugs and decide together: skip-with-ticket or assert-the-bug-fixed.

Completion criterion: a written test plan — capability × layer × oracle — the user has approved.

## 2. Implement in vertical slices

One capability at a time: write the test, run it, watch it pass against the real system, commit, next. Never write the whole plan's tests in one pass and debug them as a batch — a batch of unrun tests is a batch of guesses.

For each test, the discipline is already written down; follow it rather than improvising:

- journeys, waiting, assertions — the `writing-e2e-tests` skill
- selectors — the `locator-strategy` skill
- contract assertions — the `api-testing` skill
- data creation and ownership — the `test-data-management` skill

**Prove each test can fail.** A test you've only seen green is unverified: break the assertion (or the app) once, watch red, restore green. For bug-report scenarios, red comes free — write the test before the fix and it documents the bug.

## 3. Close the loop

Run the full set you created — plus the suite around it, at the parallelism CI uses — to catch data collisions with existing tests. Then report against the plan from step 1: each capability, its test, its layer, and anything deliberately not automated (with the reason on record).
