---
name: explore-feature
description: Run a charter-driven exploratory testing session against a feature — hunt for the failures nobody predicted, and leave findings filed, not just felt.
disable-model-invocation: true
---

# Explore Feature

Scripted tests prove the promises somebody already wrote down; an exploratory session hunts the failures nobody predicted. The failure mode this skill prevents: the aimless tour that ends in "looks fine" — a session that produces no artifact didn't happen. Every session ends in filed bugs, recorded questions, and candidate regression tests, or in a documented clean bill for a stated charter.

Read `docs/agents/testing.md` (if it exists) for environments, base URLs, and where credentials live.

## 1. Charter the session

One session, one charter: **explore [target] using [resources] to discover [information]** — "explore bulk invoice import using malformed CSVs to discover how errors surface". A time-box keeps it a session, not a wander. Where charters come from, in order of value:

- The `/analyze-story` risk register — risks whose detection point was "operational check" or an open question are exploration targets by definition.
- Where the bugs already cluster, and what changed most recently.
- The gaps a coverage look reveals: features every scripted test enters by the same door.

## 2. Attack, don't tour

Drive the real app — browser, API, or both. Alternate between the intended path and the attacks around it:

- **Interrupt** — cancel mid-save, refresh mid-flow, back button after a mutation, double-submit, kill the network and retry.
- **Starve and strain** — everything empty, then everything at maximum: longest name, largest file, most items, slowest connection.
- **Contaminate** — unicode and emoji, HTML fragments, whitespace-only, zero, negative, boundary dates (see the `generating-test-data` skill for why ugly data finds bugs).
- **Break the sequence** — steps out of order, the same action twice, the same record open in two tabs.
- **Cross the privilege line** — the URL or API call the UI hides from this role: is it hidden, or actually forbidden?

**Follow the smoke.** A slow response, a stale value, a console error, a number that's almost right — that's the trail. Deviating from the charter to chase it is the method working, not a distraction; note the detour and go.

## 3. Keep three ledgers as you go

Write findings down the moment they appear, sorted into: **bugs** (a named expectation violated — capture the repro state now, while it exists), **questions** (behavior with no spec to judge it by — for the team, not the bug tracker), and **coverage notes** (what was exercised, what wasn't reached). Sorting later loses the repro; "I'll remember it" is how sessions produce nothing.

## 4. Close with artifacts

- File each bug per the `reporting-bugs` skill — minimal steps, named oracle, evidence attached.
- Route questions to the team; ones that reveal a risk belong in the story's risk register.
- Name the candidate regressions: which findings deserve a permanent test and at which layer (the `test-pyramid` skill) — then `/automate-scenario` implements them.
- Report against the charter: what was covered, what wasn't, and whether the charter is exhausted or has earned a follow-up session.
