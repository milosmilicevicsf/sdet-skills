---
name: component-testing
description: Discipline for component and integration tests — the layer between unit and e2e. Use when testing UI component behavior (states, interactions, rendering logic) without a full browser-plus-backend journey.
---

# Component Testing

The other skills keep pointing "down a layer" — variations to the API, logic to unit tests — and leave one rung unnamed. This is it: the component test mounts a **real component in a real DOM, driven by real user events, with its data stubbed at the edge.** It proves everything the journey shouldn't have to, faster and more stably than e2e and more realistically than a unit test of a render function. Examples use Playwright/Testing Library conventions; the discipline is tool-agnostic.

Read `docs/agents/testing.md` (if it exists) for the project's component-test runner and mount setup.

## Push UI logic here, keep the journey thin

An e2e should prove one thing about a component: that it's wired into the real app. Everything else about it — loading, empty, error, and success states, conditional rendering, form-validation display, enabled/disabled logic, edge props — is provable here at a fraction of the cost and none of the flakiness. This is the layer split from the `writing-e2e-tests` and `api-testing` skills applied to the front end: one journey e2e, every UI variation as a component test.

```typescript
// GOOD: the state matrix a browser journey should never carry
test("shows a field error when the email is invalid", async ({ mount }) => {
  const form = await mount(<SignupForm />);
  await form.getByLabel("Email").fill("not-an-email");
  await form.getByRole("button", { name: "Sign up" }).click();
  await expect(form.getByText("Enter a valid email")).toBeVisible();
});
```

## Test through the user, even here

The smaller scope is not a license to reach inside. Query by role, label, and text (the `locator-strategy` ladder holds at this layer too), fire real events, and assert the rendered output. Asserting on component state, props, or instance methods couples the test to the implementation — it breaks on a refactor that changed nothing a user sees, and passes when the rendered result is wrong (the `test-review` skill's refactor and bug probes).

```typescript
// BAD: reaches into the implementation — green through a broken render
expect(wrapper.state("isOpen")).toBe(true);

// GOOD: asserts what the user sees
await expect(component.getByRole("dialog")).toBeVisible();
```

## Stub the edge, not the subject

Feed the component its dependencies — data via props or context, network via a stub at the boundary (the `network-mocking` skill) — but never mock the component under test. A test where everything around the subject is faked until only the mock's echo remains is a unit test in disguise, and a lying one.

## Independence and naming hold at every scale

Each test mounts fresh with no shared mutable module state — the same isolation law as the `test-data-management` skill, at unit scale. And the title states a capability — `disables submit until the form is valid`, not `renders correctly`; a component test named for a noun is the page tour from the `writing-e2e-tests` anti-patterns, shrunk.

## When it's the wrong layer

If proving the behavior needs the real backend or a cross-page flow, it's an API or e2e test — not a component test drowning in mocks. The signal you've gone too far: more lines of stub setup than of component behavior. Mock-heavy component tests that re-enact a journey are the expensive lie this layer exists to avoid.
