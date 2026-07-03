---
name: locator-strategy
description: Selector discipline for UI automation. Use when choosing or fixing element locators, or when tests break on markup changes.
---

# Locator Strategy

A locator is a bet on what won't change. Users find elements by role and visible text; markup, CSS, and DOM position are implementation details that churn. Bet on what the user sees, and the locator survives every refactor that doesn't change behavior.

## The ladder

Work down; stop at the first rung that uniquely matches. Each step down is a weaker bet.

1. **Role + accessible name** — `page.getByRole("button", { name: "Save" })`. The user's own view of the page; doubles as a free accessibility check. If a control has no role or name, that's an a11y bug worth raising.
2. **Label / placeholder** — `getByLabel("Email")` for form fields.
3. **Visible text** — `getByText("Order confirmed")` for non-interactive elements.
4. **Test id** — `getByTestId("cart-count")`. A contract with the app: stable by agreement, invisible to users. Use when the element has no user-facing handle (icons, containers, dynamic lists).
5. **CSS** — last resort, and only structure-free: `[data-state="open"]`, never `.sidebar > div:nth-child(3)`.

XPath with structure (`//div[3]/span`) is below the ladder — it encodes DOM position, the fastest-churning detail there is. The only defensible XPath is one expressing a relationship CSS can't, and even then prefer framework alternatives (`filter({ has: ... })`, chained locators).

## Rules

- **Uniqueness through scoping, not specificity.** When a locator matches twice, scope it to a container (`row.getByRole("button", { name: "Delete" })`) instead of piling on selectors.
- **Exact text for identity, regex for volatility.** `{ name: "Save", exact: true }` when a sibling "Save as draft" exists; `getByText(/^\d+ items?$/)` when content is dynamic.
- **One definition per element.** A locator used by more than one test lives in one place (page object, fixture, locators module) so a markup change is a one-line fix. Which pattern the project uses is in `docs/agents/testing.md` (if it exists) — follow it.
- **Never locate by style.** Classes like `.btn-primary`, `.text-red-500` change with every redesign and utility-CSS pass.

## When a locator breaks

A broken locator is information. Before patching it, ask which happened:

- **Behavior changed** (button renamed, flow redesigned) — the test must change too; update it against the new spec, don't just make the selector match again.
- **Markup changed, behavior didn't** — the old locator was too low on the ladder. Fix it by climbing *up* (role, label, testid), never by re-pinning to the new markup at the same rung.
