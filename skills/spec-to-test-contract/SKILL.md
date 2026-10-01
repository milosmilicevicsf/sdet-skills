---
name: spec-to-test-contract
description: Translate structured software specifications into deterministic test fixtures, mock data, and acceptance assertions.
---

# Spec-to-Test Contract Skill

Translates machine-readable specifications and architecture contracts into verifiable test suites.

## Core Rules
1. Map every functional requirement to at least one positive and one boundary test scenario.
2. Validate non-functional constraints (latency budgets, payload schemas) as explicit assertions.
3. Ensure zero flaky assertions by isolating mock state per test execution.
