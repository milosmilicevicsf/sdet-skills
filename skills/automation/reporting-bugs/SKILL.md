---
name: reporting-bugs
description: Discipline for filing bugs found during testing. Use when writing a bug report, or when a test failure or flaky-test diagnosis reveals a product defect.
---

# Reporting Bugs

A bug report is a **reproduction with a contract violation attached** — not a complaint, not a screenshot captioned "doesn't work". Its quality is measured by one number: how long the developer needs before they're watching the bug happen. Everything below serves that number.

## Reproduce before you report

- **Minimal steps.** Cut every step that doesn't change the outcome; each irrelevant step is a false lead the developer will chase. If you can't remove a step without losing the bug, that step is a clue — say so.
- **Owned data.** Steps start from data anyone can create ("create a project with 0 members"), never from a magic record ("log into the account where I saw it") — the same ownership law as the `test-data-management` skill.
- **A stated rate.** "Fails 3 in 10 runs" is a fact a developer can verify; "sometimes" is a shrug. For intermittent bugs, raise the rate first with the stress tricks from the `diagnosing-flaky-tests` skill — contention, throttling, repetition — and report the command that reproduces at that rate.

## Expected versus actual, with the oracle named

Expected behavior comes from a **named source** — the spec, an acceptance criterion, the previous release's behavior — quoted or linked in the report. If no source specifies the behavior, you have a question or a design gap, not a bug; file it as one, because "expected" backed only by the reporter's taste starts a debate instead of a fix.

Actual behavior is what you observed, with the evidence attached: the response body, the log line, the trace, the screenshot at the moment of failure. Evidence beats adjectives — paste the wrong value, not "the value is wrong".

## One defect, titled as its behavior

One report per defect: three symptoms of one cause is one report; two causes found in one session are two. The title states the violated behavior so the backlog reads as a list of broken promises — `declined card leaves the order stuck in "processing"`, not `checkout broken` — the same naming law the `writing-e2e-tests` skill applies to tests.

## Severity is user impact

Score by blast radius — who hits it, how often, is there a workaround, does it corrupt data — never by how hard it was to find or how annoyed you are. A cosmetic typo found after three hours of hunting is still trivial; a silent data loss found by accident is still critical.

## When a test found it

Link the failing test in the report — it's the best reproduction there is — and keep it able to go red: never weaken or retry the test to make CI green while the bug lives (the `fixing-flaky-tests` counterfeits apply). If the suite must pass meanwhile, quarantine with a link to this report, per the `triage-flaky` skill; when the fix lands, the test going green is its proof.
