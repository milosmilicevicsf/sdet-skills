---
name: analyze-story
description: Analyze a user story before implementation — testability gaps, blast radius on the rest of the system, and a ranked risk register with the cheapest detection point for each risk.
disable-model-invocation: true
---

# Analyze Story

Take a story — user story, feature proposal, change request — before or while it's refined, and answer the two questions an SDET is paid to ask early: **what could this change break, and how would we know?** The failure mode this skill prevents: quality entering the picture after the code is written, when every finding is a bug instead of a question. A risk found in refinement costs a sentence; the same risk found in production costs an incident.

Read `docs/agents/testing.md` (if it exists) so test proposals match the repo's actual frameworks and conventions.

## 1. Map the blast radius

Read the story, then explore the codebase for what the change actually touches — these are facts to find, not questions to ask:

- **Direct footprint** — the modules, endpoints, schemas, and UI surfaces the story changes.
- **Dependents** — consumers of a changed contract, shared components, features reading the same data, events and jobs triggered downstream. This is where regressions live: the story never mentions them, because the story's author wasn't thinking about them.
- **Existing state** — records created before this change that must migrate or coexist with it. New code meeting old data is a classic escape route for bugs.
- **Strained surfaces** — authorization boundaries the change crosses, concurrency it invites (two users, retries, double-submits), volume it must survive, third parties it leans on.

## 2. Grill the story

Interview the user **one question at a time** about what the story leaves unsaid; the codebase answers facts, only genuine decisions go to the humans:

- **Checkability.** Every acceptance criterion must be verifiable by a test with a definite oracle. "Works smoothly" is not checkable; "saves within 2 seconds at p95" is. Flag each criterion no test could fail.
- **Unhappy paths.** The story describes success; the incidents live in the rest — invalid input, permission denied, dependency down, concurrent edit, mid-flow abandonment. Ask which are in scope and what the specified behavior is.
- **Boundaries.** Limits and maximums, empty states, timezone and locale, first-run versus long-standing users.

A question the team can't answer yet is a finding, not a dead end — record it as an open risk rather than letting it silently resolve to "whatever gets implemented".

## 3. Rank the risks

Write the risk register. Each risk is a **falsifiable sentence** — "drafts saved before this change have no `status` field and will fail the new validation" — scored by impact × likelihood, with the blast-radius facts from step 1 keeping likelihood honest. Rank, then cut the trivia: ten sharp risks the team debates beat forty generic ones they skim.

## 4. Name the cheapest detection point

For each risk that survives ranking, name the earliest, cheapest place it would be caught:

1. **A refinement decision** — the risk dissolves if the team decides now (define the error behavior, exclude legacy data). Free; the answer becomes an acceptance criterion.
2. **A test at a layer** — the lowest layer where the failure is observable, per the `test-pyramid` skill. Most risks land at unit, component, or API; a journey earns e2e only when the risk is the wiring itself.
3. **An operational check** — some risks aren't worth a test (too rare, too environmental, low impact): a metric, an alert, or a flagged rollout is the honest answer, on record.

The proposals must hold the pyramid: name the one journey worth e2e and push every variation down. This step decides the suite's cost forever — it is the layer split `automate-scenario` will later implement.

## 5. Deliver

Produce the analysis as one document: the change and its blast radius, the open questions for refinement, the ranked risk register with each risk's detection point, and the proposed tests as capability × layer × oracle. Offer to file it where the team keeps refinement notes. When the story is implemented, `/automate-scenario` takes this analysis as its draft plan — the interview is already half done.
