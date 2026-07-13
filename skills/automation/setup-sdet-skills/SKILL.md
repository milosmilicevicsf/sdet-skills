---
name: setup-sdet-skills
description: Configure this repo for the SDET skills — record the test framework, directory layout, conventions, and CI facts these skills read. Run once before first use.
disable-model-invocation: true
---

# Setup SDET Skills

Scaffold the per-repo configuration the other skills assume: a single `docs/agents/testing.md` that answers the questions every skill otherwise has to rediscover — what framework, where tests live, how data is created, how the suite runs in CI.

This is a prompt-driven skill, not a script. Explore, present what you found, confirm with the user, then write.

## Process

### 1. Explore

Read the repo's actual state; don't assume:

- Test framework and version — `package.json` / `requirements.txt` / build files (Playwright? Cypress? Selenium? pytest? WebdriverIO?)
- Test directories and naming — existing spec files, their layout (`e2e/`, `tests/`, `*.spec.ts`?)
- Framework config — `playwright.config.ts`, `cypress.config.*`: base URLs, projects, workers, retries, reporters
- Data setup — factories, fixtures, seed scripts, API helpers already in use
- Locator conventions — do existing tests use test ids? Which attribute (`data-testid`, `data-test`, `data-cy`)? Page objects?
- Tags/annotations — `@smoke`, `@regression`, `grep` patterns in CI
- CI — workflow files that run tests: triggers, sharding, artifact upload
- `docs/agents/testing.md` — does this skill's prior output already exist?

### 2. Present and ask

Summarize what you found, then walk the user through the gaps **one section at a time** — present, get an answer, move on. Sections where exploration found a clear answer need confirmation, not interrogation.

1. **Framework and layout** — confirm what you detected; ask only about ambiguity (two frameworks present, tests scattered).
2. **Environments** — which base URLs/environments do tests run against, and where do credentials come from? (Never write secrets into the doc — record *where* they live.)
3. **Data strategy** — how should tests create data: API factories, seed scripts, transaction rollback, tagged-and-swept? (See the `test-data-management` skill for the options' trade-offs.)
4. **Locator convention** — confirm the test-id attribute and whether page objects / locator modules are the house pattern.
5. **Suites and tags** — which tags exist, what each means, which gate CI.
6. **Retry policy** — the repo's stance on retries and quarantine (the `diagnosing-flaky-tests` skill assumes retries-as-fix is forbidden; let the user consciously confirm or override).

### 3. Confirm and write

Show the draft `docs/agents/testing.md` — one section per decision above, each a few lines of fact, not prose. Let the user edit before writing.

Then add a pointer so agents find it. If `CLAUDE.md` or `AGENTS.md` exists, add (or update in place) a `## Testing` section linking to the doc. If neither exists, ask which to create — don't pick.

### 4. Done

Tell the user which skills now read this file (`writing-e2e-tests`, `component-testing`, `page-object-model`, `api-testing`, `network-mocking`, `test-data-management`, `automate-scenario`, `audit-test-suite`), and that editing `docs/agents/testing.md` directly is the normal way to change these facts — re-running this skill is only for starting over.
