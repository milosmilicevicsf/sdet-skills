---
name: network-mocking
description: Discipline for mocking, stubbing, and intercepting network in tests. Use when deciding whether to mock a dependency, or when stubbing HTTP or third-party calls in UI or integration tests.
---

# Network Mocking

Every mock trades fidelity for control: you get a fast, deterministic test and give up the guarantee that reality still looks like your fixture. That trade is worth making at the edges you don't own and ruinous at the core you're meant to be testing. The rule that follows: **mock the boundary, never the thing under test.** Examples use Playwright/TypeScript; the discipline is tool-agnostic.

Read `docs/agents/testing.md` (if it exists) for which dependencies the project mocks and where recorded fixtures live.

## Mock what you don't own

Third-party APIs, payment sandboxes with real side effects, email/SMS gateways, chronically flaky externals, and nondeterministic sources (the clock, randomness) are the legitimate targets. They're outside your control, so faking them buys determinism without hiding your own behavior.

Your own backend is not on that list. Stubbing your API inside a UI test means the test proves the front end talks to a fiction — the one thing an e2e exists to disprove. If you're mocking your backend to exercise a variation, you're testing at the wrong layer: push the variation to the API layer (the `api-testing` skill) and keep the e2e on the real stack.

```typescript
// BAD: faking your own API — the journey no longer proves the wiring
await page.route("**/api/cart", (route) =>
  route.fulfill({ json: { items: [{ sku: "A1", qty: 2 }] } }));

// GOOD: intercept the third party you don't control; hit your own API for real
await page.route("https://api.stripe.com/**", (route) =>
  route.fulfill({ json: stripeChargeSucceeded() }));
```

## A stub is a snapshot of a contract — keep it honest

A hand-written mock is frozen at the moment you wrote it; the real service moves on and the suite stays green against a fiction. This is mock drift, and it's how a fully passing suite ships a broken integration. Tie every stub to the real contract: generate it from the provider's schema, record it from a real response, or back it with a contract test that fails when the live API diverges. A stub with no link to reality is a liability with a timestamp.

## Assert the outcome, not the call

Verifying "the mock was called with X" as the *only* assertion tests your test's own wiring — it couples to implementation and passes even when the user sees nothing (the `test-review` skill's tautology and strength checks). Drive the input through the stub, then assert the user-visible or consumer-visible consequence.

```typescript
// BAD: the only assertion is that a mock ran
expect(chargeMock).toHaveBeenCalledWith({ amount: 4200 });

// GOOD: assert what the user actually gets
await expect(page.getByText("Payment confirmed")).toBeVisible();
```

## Prefer real, scope tight, clean up

Reach for a mock last, not first: a real API plus owned data (the `test-data-management` skill) beats a stub for anything in your own stack. When you do mock, treat it as per-test state — register it inside the test or fixture and let it tear down with the test, so one test's fiction never leaks into another's reality (the same isolation law as data). A route left registered across tests is pollution that fails in the suite and passes alone.
