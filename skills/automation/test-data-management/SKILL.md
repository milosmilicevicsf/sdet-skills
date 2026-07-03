---
name: test-data-management
description: Test data discipline — factories, isolation, cleanup. Use when setting up test data, fixing data collisions between tests, or designing fixtures.
---

# Test Data Management

Every law of test data follows from one principle: **a test owns its data**. It creates what it needs, nothing else can touch it, and its disappearance affects no other test. Most "flaky" suites are data-ownership violations wearing a timing costume.

## Creation

- **Create through a sanctioned channel:** the public API, or a factory that wraps it. Raw SQL inserts bypass validation and invariants, producing states the app itself could never create — tests then pass against impossible data.
- **Factories with overrides.** One factory per entity, sensible defaults, the test overrides only what it's about:

```typescript
// The test reads as: "a user with an expired card" — nothing else matters
const user = await createUser({ card: expiredCard() });
```

A test that hand-builds a 20-field object buries its intent in 19 irrelevant lines.

- **Collision-proof identity.** Any unique field gets a run-scoped unique value (`qa-${crypto.randomUUID()}@test.dev`). Hard-coded names are collisions waiting for parallelism.
- **Fixtures over setup blocks.** Push creation into the framework's fixture system (Playwright fixtures, pytest fixtures) so ownership and teardown travel with the data automatically.

## Isolation

Tests never share mutable state. Shared *readable* reference data (countries, plans, categories) is fine — it's the shared *mutable* records (the one test account everyone logs into) that serialize your suite and breed pollution.

The tell that isolation is broken: a test that fails only in the full suite, or a failure rate that moves with worker count. That's test pollution — see the `diagnosing-flaky-tests` skill.

## Cleanup

Delete what you created, and choose the mechanism by blast radius:

1. **Transaction rollback** — when tests own the database, wrap each test; nothing survives, nothing to forget.
2. **Teardown via fixture** — delete through the API in the fixture's teardown, so cleanup runs even when the test fails.
3. **Tagged data + sweeper** — in shared environments, mark everything with a recognizable prefix and run a scheduled sweep for leftovers from crashed runs.

Cleanup that only runs on success is not cleanup; a failing test is exactly when debris gets left behind.

**Exception — debuggability:** it's legitimate to keep failed-test data for inspection. Make that a deliberate policy (skip teardown on failure, rely on the sweeper), not an accident.

## Environments

Data assumptions are environment coupling. A test that "works on staging" because user 4711 happens to exist there is not automated — it's a manual test with extra steps. The test creates user 4711 or it doesn't reference it. Environment-specific facts (URLs, credentials, feature flags) live in config, read at one place; per-project conventions are in `docs/agents/testing.md` (if it exists).
