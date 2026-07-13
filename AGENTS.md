# AGENTS.md

## Cursor Cloud specific instructions

### What this repo is
This is **SDET Skills** — a documentation-only package of AI agent "skills" (see `README.md`). Each
skill is a self-contained `skills/automation/<name>/SKILL.md` Markdown file (some have supporting docs
like `anti-patterns.md` / `HTML-REPORT.md`). There is **no application, server, database, package
manifest, or build/test/lint tooling** in this repo — do not look for `package.json`, a test runner,
or CI; none exist. Changes here are edits to Markdown content.

### The one runtime dependency
The only tool involved in the workflow is the external `skills.sh` installer, run via `npx skills@latest`
(Node.js is preinstalled). It is fetched on demand by `npx`; nothing needs to be installed into the repo.

### How to validate the "product" (no build/test/lint exists)
The product works when the installer can discover and install the skills. The installer can consume this
**local working tree directly** (no need to publish to GitHub first):

- Discover/lint skills from local source: `npx --yes skills@latest add /workspace --list`
  (this parses every `SKILL.md` frontmatter and reports the `name` + `description`; a malformed skill
  shows up here).
- Full install into a throwaway project to verify end-to-end:
  ```bash
  cd $(mktemp -d) && git init -q
  npx --yes skills@latest add /workspace --agent cursor --skill '*' --copy -y
  npx --yes skills@latest list
  ```
  Skills land in `./.agents/skills/<name>/` and are tracked in `skills-lock.json`.

### Gotchas
- In an agent environment the installer detects the agent and runs **non-interactively**; when running by
  hand pass `-y`/`--all` to skip prompts so it does not hang waiting for TTY input.
- Prefer `--copy` when validating so files are copied (not symlinked) and easy to inspect.
- Each `SKILL.md` needs YAML frontmatter with `name` and `description`. User-invoked skills also set
  `disable-model-invocation: true` (e.g. `setup-sdet-skills`) — keep that flag when editing them.
