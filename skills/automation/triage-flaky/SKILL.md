---
name: triage-flaky
description: Work through a backlog of flaky tests — rank by signal damage, diagnose the worst, and leave every test either fixed, quarantined with a ticket, or deleted.
disable-model-invocation: true
---

# Triage Flaky

A session for when flakiness is a *backlog*, not a bug: CI is red-ish, nobody trusts a failure, and the list of "known flaky" tests only grows. The goal is not to fix every test today — it's that **every flaky test ends the session in a named state**, and the worst offenders get diagnosed first.

The three terminal states:

- **Fixed** — race found and removed, proven by a loop (per the `diagnosing-flaky-tests` skill).
- **Quarantined** — a tracked skip with an issue link and an owner. Visible debt.
- **Deleted** — the test's promise is covered elsewhere or wasn't worth keeping. Deletion is a legitimate outcome; say so early, because teams hoard tests.

Forbidden state: "known flaky, retried, ignored" — that's the state this session exists to drain.

## 1. Build the list from evidence

Gather the actual flake set, worst first. Prefer CI history (re-run patterns, pass-on-retry counts, failure rates over the last N runs) — ask the user for access or an export if you can't reach it. Fall back to: retry configuration, `skip`/`flaky` annotations, and the user's own shortlist.

Rank by **signal damage**: failure rate × how often the test runs × whether it gates merges. A 20%-flaky test on every PR outranks a 60%-flaky nightly.

## 2. Present the docket, agree the budget

Show the ranked list with the evidence per test. Agree with the user: how many will we diagnose properly this session (deep work), and what happens to the rest (quarantine now, docket for later). An explicit budget prevents the failure mode of half-diagnosing everything and finishing nothing.

## 3. Drain the docket

For each test inside the budget, run the `diagnosing-flaky-tests` loop: reproduce at a stated rate, name the race, fix, prove with the loop. Batch the cheap ones: if the scan shows ten tests sharing one cause (same shared account, same sleep pattern), that's one diagnosis and one mechanical fix across ten tests.

For each test outside the budget: quarantine — skip annotation, issue with the evidence gathered in step 1, owner. Never a bare skip.

If a diagnosis reveals a product bug (the race is in the app), file it as such and leave the test red-capable — a test that catches a real race is the suite working.

## 4. Report

End with the ledger: every test from the docket and its terminal state (fixed / quarantined+ticket / deleted), the causes found (patterns count — three "shared account" findings is a systemic finding), and what the recurrence prevention is: which convention from `docs/agents/testing.md` (or the `test-data-management` / `writing-e2e-tests` skills) would have prevented each cause, and whether the team is following it.
