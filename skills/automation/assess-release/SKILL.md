---
name: assess-release
description: Assess whether a release candidate is safe to ship — what changed, what proves each change, what debt rides along — ending in a go/no-go with every residual risk named.
disable-model-invocation: true
---

# Assess Release

Answer "can we ship this?" with evidence. The failure mode this skill prevents is **green-build worship**: CI green means the tests that exist passed — it says nothing about changes with no test, tests that lie, or the quarantine list absorbing failures. The deliverable is never a naked yes: it's a recommendation plus every residual risk, named, with its detection plan. A risk shipped knowingly with a rollback path is a decision; one nobody named is an incident.

Read `docs/agents/testing.md` (if it exists) for which suites and tags actually gate a release.

## 1. What changed

Build the change list since the last release — commits, PRs, stories — and group it by risk, not by author: schema migrations, changed contracts with consumers, authorization changes, dependency bumps, and everything touching money, data deletion, or messaging sit at the top. This is the `analyze-story` blast-radius lens applied to the whole candidate; where an analysis exists for a story, its risk register is input here.

## 2. What proves each change

For every change group, name the evidence: which tests, at which layer (the `test-pyramid` skill), or which exploratory session (`explore-feature`). Two findings fall out mechanically — changes whose evidence is **nothing**, and changes whose only evidence is a suite known to lie (retried to green, asserting nothing). Run the gating suite at CI parallelism if it hasn't run against the exact candidate; evidence from a different build is evidence about a different release.

## 3. What rides along

The debt that ships with the code:

- **Quarantined and skipped tests** that overlap changed areas — a quarantined test is a hole in the gate, acceptable only while someone can say what covers that hole today (the `triage-flaky` ledger is the source).
- **Open bugs** by severity in changed areas, severity per the `reporting-bugs` skill — impact, not annoyance.
- **Suite trust**: retry configuration and recent flake rate. A gate that passes on `retries: 2` filters nothing; say so rather than counting its green.

## 4. The judgment

Deliver the verdict in one page: **go or no-go**, the evidence table (change group × proof), and the residual-risk list — each risk with its detection in production (the metric, alert, feature flag, or rollback trigger that catches it if it fires). A no-go names the shortest path to go: which evidence gap, closed by which test or session. Offer to file the gaps as tracked issues.
