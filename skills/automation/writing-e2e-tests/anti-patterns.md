# E2E Anti-Patterns

Each entry: the pattern, the tell, the fix.

## The page tour

A test that navigates and asserts "the page rendered" without exercising a behavior.

```typescript
// BAD: proves nothing a user cares about
test("dashboard works", async ({ page }) => {
  await page.goto("/dashboard");
  await expect(page.locator("h1")).toBeVisible();
});
```

The tell: the test name is a noun ("dashboard"), not a capability. The fix: name the journey, or delete the test — a smoke test is fine, but call it one and keep exactly one.

## The sleep chain

```typescript
// BAD: every sleep is a guess
await page.click("#next");
await page.waitForTimeout(2000);
await page.click("#confirm");
await page.waitForTimeout(5000);
```

The tell: `waitForTimeout`, `sleep`, `Thread.sleep`, `cy.wait(3000)`. The fix: wait on a signal — element state, network response, URL. If no signal exists, that's an app gap to raise, not paper over.

## Conditional flow

```typescript
// BAD: the test adapts to whatever the app does
if (await page.locator(".modal").isVisible()) {
  await page.click(".modal .close");
}
```

A test with `if` branches doesn't know what state the app is in — which means it isn't controlling its preconditions. The tell: `if`/`try-catch` around interactions. The fix: pin the precondition (seed data, control feature flags) so exactly one path exists.

## The shared account

Tests logging into the same `test@example.com` account trample each other's state and forbid parallelism. The fix: per-test users created via API/fixture — see the `test-data-management` skill.

## Asserting internals

```typescript
// BAD: user can't see a Redux store
expect(await page.evaluate(() => window.store.getState().cart.items)).toHaveLength(2);

// GOOD: user sees the cart badge
await expect(page.getByTestId("cart-count")).toHaveText("2");
```

The tell: `page.evaluate` reaching into app state, asserting on request payloads mid-journey, checking CSS class names. The fix: assert the user-visible consequence.

## The mega-test

One test covering login + browse + cart + checkout + refund. When it fails at step 40, the first 39 steps are suspects. The fix: one journey per test; share the common prefix via setup (API login, storage state), not via test length.

## Retry as a fix

Marking a flaky test `retries: 3` converts a red signal into a slow green lie. Retries are transport-level insurance, not a treatment for nondeterminism — see the `diagnosing-flaky-tests` skill.
