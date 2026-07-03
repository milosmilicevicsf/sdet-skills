---
name: api-testing
description: Discipline for API-level automated tests. Use when writing tests against HTTP/REST/GraphQL endpoints, or deciding what to cover at the API layer instead of the UI.
---

# API Testing

The API layer is where most behavior should be tested: faster than e2e, more realistic than unit tests, and immune to UI churn. Test the **contract** — what a consumer can rely on — through real HTTP calls against a running service. Examples use Playwright's `request` fixture; the discipline is tool-agnostic.

When exploring the codebase, read `docs/agents/testing.md` (if it exists) for base URLs, auth setup, and environment conventions.

## Push tests down to this layer

Before writing an e2e test, ask: does this behavior need a browser to prove? Validation rules, permissions, edge cases, error codes, pagination — all provable here at a fraction of the cost. The UI test then only needs the happy path that proves the wiring. One journey e2e, every variation at the API layer.

## Assert the contract, tolerate the rest

A consumer relies on: status code, the fields it reads, error shape, side effects. Assert those. Don't assert field-by-field equality on the entire payload — a new optional field is not a regression, and a test that fails on one is crying wolf.

```typescript
// BAD: breaks when any field is added
expect(body).toEqual({ id: 42, name: "Alice", createdAt: "..." });

// GOOD: asserts what consumers rely on
expect(response.status()).toBe(201);
expect(body).toMatchObject({ name: "Alice" });
expect(body.id).toEqual(expect.any(Number));
```

For structural coverage, validate against the schema (OpenAPI, zod, JSON Schema) — one schema assertion replaces fifty field checks and stays in sync with the source of truth.

**Test the unhappy paths deliberately.** Error contracts are contracts: wrong input → 400 with a usable error body, missing auth → 401, forbidden → 403, absent resource → 404. Consumers build retry and error UI against these; an untested error shape breaks them silently.

## Verify through the interface

Prove a write happened by reading it back through the API — not by querying the database. A database assertion couples the test to storage internals and skips the read path, which is half the contract.

```typescript
// GOOD: the read path confirms the write
const created = await request.post("/api/projects", { data: { name: "Q3 Plan" } });
const { id } = await created.json();
const fetched = await request.get(`/api/projects/${id}`);
expect((await fetched.json()).name).toBe("Q3 Plan");
```

Reach into the database only when the API deliberately hides the effect (audit logs, async side effects with no read endpoint) — and treat each such reach as a flag that the contract may be missing a surface.

## Independence and data

Same law as e2e: every test creates what it needs and runs in any order, in parallel. Create via the API itself, make names collision-proof, clean up owned resources — the full discipline is in the `test-data-management` skill.

Auth is setup, not journey: acquire tokens once per worker via a fixture, not per-test login calls. Testing the auth flow itself is its own test, not every test's preamble.
