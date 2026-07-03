---
name: writing-e2e-tests
description: Discipline for writing UI end-to-end tests. Use when writing or editing e2e/UI tests, or when the user mentions Playwright, Cypress, or Selenium specs.
---

# Writing E2E Tests

An e2e test is a **user journey**: a real person trying to accomplish something through the UI. Every rule below follows from that one idea. Examples use Playwright/TypeScript; the discipline is tool-agnostic.

When exploring the codebase, read `docs/agents/testing.md` (if it exists) for the project's framework, directory layout, and tag conventions.

## Journeys, not page tours

A test earns its e2e cost by covering a journey a user actually takes — "user can check out with a saved card" — not by visiting pages and asserting they render. If a behavior can be verified at a cheaper layer (unit, API), test it there; e2e is reserved for what only the full stack can prove. Before writing, name the journey and the layer: if you can't say why this must be e2e, it shouldn't be.

## Independence

Every test runs alone, in parallel, in any order, on a fresh browser context. No test reads state another test wrote. The tell: a test that passes alone but fails in the suite (or vice versa) is order-dependent — fix the dependency, never the order.

Get to the starting state through the fastest reliable channel — API calls, seeded fixtures, storage state — not by clicking through the UI. UI navigation is the journey under test, not the setup mechanism. See the `test-data-management` skill for data isolation.

## Waiting

**Never sleep.** A hard-coded wait is either too short (flaky) or too long (slow) — usually both, on different days. Wait on a **signal**: an element state, a network response, a URL change.

```typescript
// BAD: guessing how long the app needs
await page.waitForTimeout(3000);
await page.click("#submit");

// GOOD: web-first assertion retries until the signal arrives
await expect(page.getByRole("button", { name: "Submit" })).toBeEnabled();
await page.getByRole("button", { name: "Submit" }).click();
```

Modern frameworks auto-wait on actions and retry assertions — lean on that. If you feel the need for a sleep, the app is missing a signal; find it or expose it.

## Assertions

Assert what the **user can see**: text, visible state, URL — not internal wiring (network payloads, store contents, CSS classes). One journey per test, but assert every user-visible checkpoint along it; a journey that only asserts at the end hides where it broke.

Expected values come from an independent source of truth — the spec, a known literal — never recomputed the way the app computes them.

## Naming

The title states the behavior, readable as a specification: `user can retry a declined payment`, not `test checkout flow 2`. If you can't title the behavior, you don't know what you're testing yet.

## Anti-patterns

See [anti-patterns.md](anti-patterns.md) for the catalogue with examples: page tours, sleep chains, conditional flow, shared accounts, over-asserting internals, and the mega-test.

For selector discipline, see the `locator-strategy` skill. When a test you wrote is intermittently red, see the `diagnosing-flaky-tests` skill — don't add retries.
