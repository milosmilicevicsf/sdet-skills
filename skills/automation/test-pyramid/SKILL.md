---
name: test-pyramid
description: The cost model behind test layers. Use when deciding at which layer a behavior should be tested, when planning coverage for a story or feature, or when e2e tests are proliferating.
---

# Test Pyramid

The pyramid is a **cost model, not a dogma about shape**: every layer up the stack multiplies what a test costs, so each behavior is proven at the lowest layer that can prove it — and the count per layer falls out of that decision, never a quota to fill. A suite that inverts this doesn't fail loudly; it decays: slower every month, flakier every quarter, until a red build means "re-run it" instead of "something broke".

## What each layer alone can prove

The decision per behavior is one question: **what is the lowest layer where this behavior's failure is observable?**

- **Unit** — pure logic: calculation, parsing, branching. Milliseconds, zero flake surface.
- **Component** — UI states and rendering logic on a real DOM, data stubbed at the edge (the `component-testing` skill).
- **API** — behavior as a consumer contract: validation, permissions, error shapes, persistence (the `api-testing` skill).
- **E2E** — the one thing only it proves: the real parts are wired into a journey a user can complete (the `writing-e2e-tests` skill).

Hence the standing rule across these skills: **one journey end to end, every variation at a lower layer.** An e2e per validation message pays browser prices for API answers.

## Where the cost actually is

Runtime is the visible cost and the smallest one. The compounding costs:

- **Flake surface** — every real dependency in the chain (browser, network, backend, data) is one more place for a race; the failure odds compound per moving part. Lower layers stub exactly the parts they don't prove.
- **Diagnosis time** — a unit failure names the function; an e2e failure names a suspect list four systems long. Paid on every failure, including the false ones.
- **Maintenance** — an e2e crosses everything, so anything's change can break it. The more tests live high, the more every refactor costs.

## Quantity is a symptom

Read counts as a diagnostic, never as a target. An inverted pyramid (300 e2e, 12 API tests) means layer decisions are being defaulted, not made — usually because e2e is the only harness the team has, which is an infrastructure gap worth naming as its own finding.

Two consequences teams resist:

- **An e2e must pay rent.** It names its journey, states why no lower layer proves it, and holds the `writing-e2e-tests` bar. A flaky e2e is not partial coverage — it's negative coverage: it burns CI minutes and trains people to ignore red.
- **Deletion is a win.** A duplicated journey, or an e2e provable at the API layer, moves down or goes. Fewer, truer tests gate a release better than many doubtful ones.

## The pyramid is the middle of a longer ladder

Shift left: a bug's cost grows with its distance from the keystroke that caused it. The full detection ladder runs from a refinement question (free — the `analyze-story` skill), through types and contract checks, up the pyramid, out to production monitoring — the honest home for risks too rare or too environmental to rehearse in CI. For each risk, name the earliest rung that catches it; a test is the middle of the ladder, not the only tool on it.
