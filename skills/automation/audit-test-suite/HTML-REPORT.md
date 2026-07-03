# HTML Report Format

The audit is rendered as a single self-contained HTML file in the OS temp directory — nothing lands in the repo. Tailwind comes from CDN; no build step, no dependencies.

## Where to write it

Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows). Write to `<tmpdir>/test-suite-audit-<timestamp>.html` so each run gets a fresh file.

Open it for the user — `xdg-open <path>` on Linux, `open <path>` on macOS, `start <path>` on Windows — and tell them the absolute path.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Test suite audit — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
  </head>
  <body class="bg-slate-950 text-slate-100 font-sans antialiased">
    <main class="max-w-3xl mx-auto px-6 py-10 space-y-8">
      <header>...</header>
      <section id="stats">...</section>
      <section id="verdict">...</section>
      <section id="findings">...</section>
      <section id="top-three">...</section>
      <footer>...</footer>
    </main>
  </body>
</html>
```

Dark theme by default — the verdict box reads better on dark, and it photographs well for sharing.

## Header

- Skill badge: `/audit-test-suite` (small pill, blue tint)
- Title: `Test Suite Audit — {{repo or project name}}`
- Meta line: framework detected, test count, audit mode (`static analysis` or `CI history included`), date

No intro paragraph — straight into the numbers.

## Stats row

Four compact stat cards in a grid. Pick the four that tell the headline story for *this* suite — don't pad with zeros:

| Typical stat | When to show |
|---|---|
| e2e / API / unit counts | always |
| `retries: N` (suite-wide) | when config sets retries > 0 |
| sleep call count | when any `waitForTimeout` / `sleep` found |
| `workers: 1` | when parallelism is disabled (often a symptom) |
| skipped / quarantined count | when `skip`/`fixme`/`flaky` annotations exist |

Use red/orange accent on stats that are themselves findings (e.g. `retries: 3` in red).

## Verdict

One `<section>` with a red left border and tinted background. Label: **VERDICT** (uppercase, small). Body: the one-paragraph verdict from step 3 of the skill — can this suite gate releases, and the single biggest reason.

If the suite is trustworthy, use green tint instead of red. Most audits won't be.

## Findings

Grouped by severity in this order: **Trust → Stability → Cost → Coverage**.

Each finding is one row in a list:

- **Severity label** — left column, colour-coded: Trust = red, Stability = amber, Cost = purple, Coverage = slate
- **Title** — pattern name (e.g. "Retry-as-fix, suite-wide")
- **Location** — monospaced file:line, blue tint
- **Body** — one or two sentences: what's wrong and the direction of the fix

Group repeated instances — one finding per pattern with a count ("10 structure-pinned locators across 2 files"), not one row per occurrence.

Cap at what's actionable. If there are 400 sleep calls, that's one finding with `400×` in the title, not 400 rows.

## Top three

Numbered list at the bottom, on a slightly darker background. Each item: bold lead phrase, then one sentence of why this ranks in the top three.

## Footer

Two items, small grey text: `sdet-skills · audit-test-suite` and the repo path or name being audited.

## What not to include

- No JavaScript beyond Tailwind CDN — the report is static.
- No files written inside the audited repo (no `reports/`, no `docs/`).
- No line-coverage percentages — this audit is about behavior trust, not coverage metrics.
- No Mermaid — test-suite findings are lists, not graphs.
