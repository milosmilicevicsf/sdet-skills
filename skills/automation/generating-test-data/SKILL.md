---
name: generating-test-data
description: Generating data at dataset scale — seed data, synthetic data, bulk and production-derived datasets. Use when seeding an environment, generating realistic or large datasets, or tempted to copy production data.
---

# Generating Test Data

The `test-data-management` skill governs the record one test owns; this skill governs data at **dataset scale** — the seed an environment starts from, the thousand orders a performance test needs, the "realistic" accounts a demo or exploratory session runs against. One principle rules all of it: **a dataset is code** — generated, versioned, rebuilt from scratch with one command. A hand-curated database nobody can recreate is an outage with a delay on it.

Read `docs/agents/testing.md` (if it exists) for the repo's factories, seed scripts, and environment conventions.

## Boring data finds nothing

A thousand copies of `John Smith, john@test.com, $10.00` exercise one path a thousand times. Bugs live where the data is ugly, so put the ugliness in **deliberately**: names with unicode, apostrophes, and length extremes; quantities at zero, negative, and the documented maximum; dates on month ends, leap days, and timezone boundaries; optional fields absent, not defaulted. And shape matters as much as values — real systems are skewed (a few enormous accounts, a long tail of empty ones), and pagination, sorting, and timeout bugs only show against that skew, never against uniform rows.

## Random, but reproducible

Generate with seeded randomness — a faker with a fixed seed per dataset version — so two runs of the generator produce the same data and a failure against it reproduces. Unseeded random data is a flaky suite by design: the failing value vanishes on the next generation, taking the bug with it.

## Production data is a liability, not a shortcut

A production copy carries PII and legal exposure, goes stale the day it's taken, and still re-identifies people by combination after naive masking. Prefer synthesizing with production's **shape**: measure the distributions, volumes, and edge frequencies from prod, then generate fresh data that matches them — realism without a single real person in it. If a real copy is genuinely unavoidable, anonymize irreversibly through a sanctioned pipeline, and prove the properties tests depend on survived it: uniqueness still unique, foreign keys still joined, formats still valid.

## Seed data is repo code

The baseline an environment needs — reference tables, roles, plans, the demo tenant — lives in the repo and applies **idempotently**, so running it twice is safe and rebuilding from empty is routine. The acceptance test for any seed: a teammate recreates the environment from scratch with one command. Seeds hold the shared *readable* baseline; records a test mutates are created by that test, per the `test-data-management` skill — a seed that tests write to is a shared-state violation with extra steps.

## Bulk data still obeys the invariants

Volume tempts a shortcut: hand-written SQL rows that skip validation and produce states the app could never create — the same trap the `test-data-management` skill names, at scale. Generate every record through the factory logic that encodes the invariants; if the API is too slow for millions of rows, keep the generation sanctioned and move only the *loading* to the bulk path. And state the dataset's contract next to it: what a performance run may assume (row counts, distributions), so the numbers mean the same thing run to run.
