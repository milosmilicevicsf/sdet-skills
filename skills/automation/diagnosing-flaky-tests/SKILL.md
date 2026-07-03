---
name: diagnosing-flaky-tests
description: Diagnosis loop for flaky tests. Use when a test fails intermittently, passes on retry, or fails only in CI.
---

# Diagnosing Flaky Tests

A flaky test is a bug with a reproduction rate below 100%. Never treat it with retries, re-runs, or longer timeouts — those convert a red signal into a slow green lie. Diagnose it like any other bug: raise the reproduction rate until it's debuggable, then find the race.

## Phase 1 — Raise the reproduction rate

You cannot debug a 1-in-50 failure. Make it fail on demand before theorizing:

1. **Loop it.** Run the single test 50–100× (`npx playwright test path/to/spec.ts --repeat-each=50`). Record the failure rate — this number is your feedback loop.
2. **Add contention.** Flakiness often only appears under parallel load: run with full workers, or alongside the suite.
3. **Starve it.** CI machines are slower than laptops. Throttle CPU/network (`--cpu-throttling-rate`, browser network emulation, `docker run --cpus=0.5`) to widen timing windows.
4. **Reorder.** If it only fails after certain tests, it's order-dependent — bisect the preceding set until the polluting test is found.

**Completion criterion:** one command that fails at a rate you state (e.g. "12/50 runs"). No rate, no Phase 2. If 500 runs under stress produce zero failures locally, capture evidence from where it does fail — CI traces, videos, screenshots — and debug from the artifact instead.

## Phase 2 — Read the failure, name the race

Capture one failing run with full artifacts (trace, video, DOM snapshot at failure). Then locate the disagreement: what did the test assume had already happened that hadn't?

Flakiness is nearly always one of these; check in order of frequency:

- **Async race** — asserting before a network response, animation, or render settles. The tell: failure screenshot shows a spinner or stale content.
- **Test pollution** — shared state (account, database rows, storage) mutated by another test. The tell: fails in the suite, passes alone; failure rate changes with worker count.
- **Data collision** — two parallel runs grabbing the same record. The tell: "already exists", deadlocks, 409s.
- **Time dependence** — assumes a timezone, date boundary, or "fast enough" machine. The tell: fails at midnight, month-end, or only in CI.
- **Infrastructure** — CI runner variance, third-party outages. Real, but blame it last: infrastructure is where undiagnosed races hide.
- **App bug** — the race is in the product, not the test. A flaky test that catches a real race is doing its job; file the bug, don't silence the test.

State the race as a falsifiable sentence: "the test clicks Submit before the cart POST resolves; if I wait on the response, the failure rate drops to 0."

## Phase 3 — Fix and prove

Fix the race itself: wait on the missing signal, isolate the data, pin the clock. Forbidden fixes: `waitForTimeout`, `retries`, marking it `flaky`/`skip`, reordering tests to hide pollution.

**Prove it with the Phase 1 loop:** the same command at the same stress level, 0 failures over at least as many runs as reliably reproduced it before. A fix you haven't looped is a guess.

If the fix is genuinely out of reach (vendor bug, platform issue), quarantine explicitly — a tracked skip with an issue link — so the suite stays trustworthy. A quarantined test is a debt with a ticket; a retried test is a debt nobody can see.
