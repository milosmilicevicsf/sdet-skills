---
name: test-review
description: Review automated tests for value and trustworthiness. Use when reviewing test code in a PR, or when the user asks whether tests are worth keeping.
---

# Test Review

Production code review asks "does it work?". Test review asks two different questions: **would this test catch the bug it exists for**, and **will it lie to us** — fail when nothing is wrong, or pass when something is. Review every test in the diff against all four axes below; a test that fails an axis gets a named finding, not a vague "could be better".

## Axis 1 — Does it test behavior?

The test must fail if the behavior breaks, and survive if only the implementation changes. Run the thought experiment both ways:

- *Refactor probe:* imagine renaming internals, swapping a library, restructuring the DOM. Would this test break? If yes — implementation-coupled. Common tells: asserting on mocks of internal collaborators, structure-pinned selectors, DB queries verifying API writes.
- *Bug probe:* imagine the feature silently breaking. Would this test go red? If no — it's decoration. Common tells: page tours (navigate + "is visible"), tests with no assertion on the outcome, asserting only that no error was thrown.

## Axis 2 — Can the assertion lie?

- **Tautology check:** does the expected value come from an independent source (spec, known literal), or is it recomputed the way the code computes it? `expect(total(items)).toBe(items.reduce(...))` passes by construction, forever.
- **Vacuity check:** can the test pass while asserting nothing? Assertions inside `if` blocks or loops over possibly-empty collections silently skip; a journey asserting only at the end hides where it broke.
- **Strength check:** `toBeVisible()` where the content matters, `toBeTruthy()` where the value matters, `not.toThrow()` as the only assertion — each passes for a thousand wrong reasons. The assertion should pin what the spec pins.

## Axis 3 — Will it hold up in the suite?

- Owns its data (unique identity, sanctioned creation, teardown that runs on failure) — violations per the `test-data-management` skill.
- No sleeps, no conditional flow, no retry-as-fix — violations per the `writing-e2e-tests` anti-pattern catalogue.
- Locators bet on behavior, not markup — per the `locator-strategy` ladder.
- Right layer: an e2e test proving something an API test could prove is a cost finding, not a style nit.

## Axis 4 — Does it read as a specification?

The title states a capability (`user can retry a declined payment`); the body reads as arrange–act–assert without scrolling; a reader can tell *what the product promises* from the test alone. Helpers and page objects are fine — indirection that hides the assertion is not.

## Reporting

Group findings by severity, each tied to a file/line and named for its pattern:

- **Trust** — the test can lie (tautology, vacuous assertion, retry-as-fix, pollution risk). These make the suite worse than no suite; block on them.
- **Cost** — the test will break for wrong reasons or burn time (implementation coupling, wrong layer, brittle locators, sleeps).
- **Clarity** — naming, structure, intent.

For each finding, state the pattern, why it bites, and the one-line direction of the fix. A finding without a direction is a complaint.
