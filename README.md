# gh-as-code

Shared GitHub Actions for the `posidon16` projects (DRADIS, Orion, ...). Project repos call the reusable
workflows here instead of copying YAML, so CI stays consistent and an improvement lands everywhere at once.

## Contents

| Path | What it is |
| --- | --- |
| `.github/workflows/docs-site.yml` | Reusable: pnpm install, lint, types, build, optional output checks, artifact upload |
| `.github/workflows/repo-hygiene.yml` | Reusable: no tracked `.env` files; optional `meta.json`-per-section check |
| `.github/workflows/lint.yml` | Runs `actionlint` on this repo's workflows for every PR |
| `examples/` | Caller `ci.yml` files (`<sha>` is a placeholder; not run from here) |
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

- **This repo is public** so Dependabot can read it. For a private repo Dependabot's `github-actions` updates
  fail with `git_dependencies_not_reachable`, so the pins in callers never get bump PRs. Keep it free of
  anything project-specific or secret.
- Pin callers to a tag or commit SHA, not `@main`, once one exists. Cut tags (`v1`, `v1.1`) when a change is
  ready to roll out; Dependabot's `github-actions` ecosystem then proposes a bump PR in each caller, and that
  PR's CI runs the new version first.
- Third-party actions inside the workflows are SHA-pinned with the tag in a comment. Keep that convention.
- A change here affects a caller only once it bumps its pin. `lint.yml` catches syntax and expression errors on
  PRs but not behaviour changes; for those, point one caller at `@<branch>` before tagging.
- A reusable workflow cannot set its own `concurrency` for the caller or define the caller's triggers; those
  stay in each repo's `ci.yml`.
