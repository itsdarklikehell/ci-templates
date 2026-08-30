# ci-templates

Reusable GitHub Actions CI workflows for the `itsdarklikehell` fleet.

Single source of truth for lint / typecheck / test / build across all repos.
Each repository gets a *thin* caller (`.github/workflows/ci.yml`) that `uses:`
one of these workflows — so fleet-wide CI changes happen here, once.

## Workflows

| Workflow | Use for | Ref |
|----------|---------|-----|
| `ci-node.yml` | Node.js / TypeScript (`package.json`) | `itsdarklikehell/ci-templates/.github/workflows/ci-node.yml@v1` |
| `ci-python.yml` | Python (`pyproject.toml` / `setup.py` / `requirements.txt`) | `...ci-python.yml@v1` |
| `ci-go.yml` | Go (`go.mod`) | `...ci-go.yml@v1` |
| `ci-generic.yml` | Dockerfile / shell scripts only | `...ci-generic.yml@v1` |

## Per-repo caller (copy into `<repo>/.github/workflows/ci.yml`)

```yaml
name: CI
on:
  push:
    branches: [main, master]
  pull_request:
  workflow_dispatch:
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
permissions:
  contents: read
jobs:
  ci:
    uses: itsdarklikehell/ci-templates/.github/workflows/ci-node.yml@v1
    with:
      node-version: "20"
```

Swap `ci-node.yml@v1` for `ci-python.yml@v1` (input `python-version: "3.12"`)
or `ci-go.yml@v1` (input `go-version: "1.22"`) as appropriate.

## Design principles

- **Safe on first rollout.** Lint/test/build steps only execute when the repo
  actually has the corresponding script or config, so enabling CI does not
  instantly paint 30 repos red.
- **Config-gated enforcement.** `ruff`/`mypy` run *only* when the repo has opted
  in via a config file. Repos without those tools are skipped, not failed.
- **Least privilege.** Each job requests `contents: read` only.
- **Concurrency.** Callers cancel superseded runs on the same ref.
- **Tagged, not floating.** Repos pin `@v1`; template changes are promoted here
  and re-tagged after validation.

## Updating the fleet

1. Edit a workflow here, push to `main`.
2. Validate on a pilot repo's PR.
3. Bump the `v1` tag (`git tag -f v1 && git push -f --tags`) once green.
4. Every caller referencing `@v1` picks up the change automatically.

## Versioning

- `@v1.0.0` — **immutable anchor.** Points at the validated baseline (lint/typecheck/test/build, config-gated). Pin callers to this for stability.
- `@v1` — floating bootstrap tag tracking `v1.0.0`. It moved during initial rollout debugging; it now sits at `v1.0.0` and will only advance on a deliberate, pilot-validated change. New callers should prefer `@v1.0.0`.
- Future breaking changes ship as `@v2`, `@v3`, … so old callers keep working.

### Rollout status (2026-08-30)

- 15 repos: CI added via thin caller, merged, green on default branch.
- 9 repos: already shipped their own `.github/workflows/ci.yml` — left intact, not clobbered.
- 9 repos: CI PR opened but red — genuine repo defects (lockfile drift, removed `distutils`, `mcp` v2 break, real lint/test errors). CI is working as intended; these are owner fixes, not template bugs.
