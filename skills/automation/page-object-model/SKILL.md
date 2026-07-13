---
name: page-object-model
description: Discipline for structuring UI test code with page objects. Use when creating or refactoring page objects, or auditing whether an existing page object layer helps or hurts.
---

# Page Object Model

A page object is a **behavior API for a screen** — it exposes what a user can *do* there and hides how the DOM is wired. The failure mode it prevents is duplication and churn: the same selectors and click sequences copy-pasted across fifty tests, so one markup change breaks all fifty. The failure mode it *causes* when done wrong is worse: a layer of indirection that hides the assertion and reads as ceremony. Examples use Playwright/TypeScript; the discipline is tool-agnostic.

Read `docs/agents/testing.md` (if it exists) for whether the project uses page objects, screenplay, or a locators module — follow the house pattern instead of introducing a rival one.

## A page object exposes capabilities, not selectors

The methods are the verbs a user has on the screen — `signIn(email, password)`, `addToCart(sku)` — each returning either the data the user now sees or the next page object. A class that only exposes getters for locators is a selector bag with extra syntax; it moves the duplication, it doesn't remove it.

```typescript
// BAD: a selector bag — every test still writes the flow itself
class LoginPage {
  emailInput = () => this.page.getByLabel("Email");
  passwordInput = () => this.page.getByLabel("Password");
  submitButton = () => this.page.getByRole("button", { name: "Sign in" });
}

// GOOD: a capability — the flow lives once, tests read as intent
class LoginPage {
  async signIn(email: string, password: string): Promise<DashboardPage> {
    await this.page.getByLabel("Email").fill(email);
    await this.page.getByLabel("Password").fill(password);
    await this.page.getByRole("button", { name: "Sign in" }).click();
    return new DashboardPage(this.page);
  }
}
```

## Keep the assertion in the test

The page object provides *state*; the test *asserts* it. An `expectLoggedIn()` method buried in the object hides the promise — a reader of the test sees method calls and can't tell what the product guarantees (the `test-review` skill's fourth axis). Expose `heading()` or `errorMessage()` and let the test hold the oracle. The one thing a method may legitimately do before returning is wait on its *own* completion signal (per the `writing-e2e-tests` skill), so callers don't race — that is a wait, not a spec assertion.

## One definition per element, composed not inherited

Each locator lives in exactly one place — the page object — so a markup change is a one-line fix (the `locator-strategy` skill's rule), and the locators there are built down the same ladder; a page object full of `.nth(3)` CSS is a brittle layer wearing an abstraction. Model shared widgets — header, data table, modal — as their own components and compose them, rather than growing a `BasePage` every screen inherits. Deep inheritance couples unrelated screens: a rename on `BasePage` ripples everywhere.

## Model by reuse, not by page count

Not every route needs an object. A page object with a single caller is premature abstraction — inline it until a second test wants the same flow. The layer earns its keep when a capability is shared or a screen is genuinely complex, not as a reflex for every URL.

## Auditing an existing layer

When reviewing or refactoring page objects, the tells of a layer that hurts — and the direction of the fix:

- **Selector bags** (only locator getters, no behavior) → convert to capability methods; move the repeated flow into the object.
- **Hidden assertions** (tests are just lists of `expectX()` calls) → lift assertions into the tests; expose state instead.
- **Leaked `page`** (tests still call `page.click` alongside the object) → the capability is missing; add it, stop reaching around the layer.
- **God `BasePage`** (one base class, dozens of unrelated helpers) → split into composable components.
- **`void` returns that force re-location** (tests re-find the next screen's elements) → return the next page object.
- **Structure-pinned locators inside** → climb the `locator-strategy` ladder in the one place, fixing every test at once.
