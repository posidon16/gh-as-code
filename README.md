# gh-as-code

Shared GitHub Actions for the `posidon16` projects (DRADIS, Orion, ...). Project repos call the reusable
workflows here instead of copying YAML, so CI stays consistent and an improvement lands everywhere at once.

## Contents

| Path | What it is |
| --- | --- |
| `.github/workflows/docs-site.yml` | Reusable: pnpm install, lint, types, build, optional output checks, artifact upload |
| `.github/workflows/repo-hygiene.yml` | Reusable: no tracked `.env` files; optional `meta.json`-per-section check |
| `examples/` | Caller `ci.yml` files for Orion and DRADIS (not run from here) |
| `templates/` | Files that cannot be shared by reference and are copied into each repo (Dependabot) |

## Using a workflow

```yaml
jobs:
  docs:
    uses: posidon16/gh-as-code/.github/workflows/docs-site.yml@<tag-or-sha>
    with:
      required-outputs: |
        out/llms.txt
```

`docs-site.yml` inputs: `working-directory` (default `docs`), `required-outputs`, `artifact-name`,
`artifact-path`, `retention-days`. It expects `<working-directory>/package.json` (with `packageManager`),
`pnpm-lock.yaml` and `.nvmrc`, and the scripts `lint`, `types:check` and `build`.

## Setup notes

- **This repo is private.** Other private repos can only call its workflows after enabling
  Settings > Actions > General > Access > "Accessible from repositories owned by the user".
- Pin callers to a tag or commit SHA, not `@main`, once one exists. Cut tags (`v1`, `v1.1`) when a change is
  ready to roll out; Dependabot's `github-actions` ecosystem bumps the reference.
- Third-party actions inside the workflows are SHA-pinned with the tag in a comment. Keep that convention.
- A change here affects every caller on the next run once they bump. Test changes on a branch by pointing
  one caller at `@<branch>`.
- A reusable workflow cannot set its own `concurrency` for the caller or define the caller's triggers; those
  stay in each repo's `ci.yml`.
