# ci-templates

Reusable GitHub Actions CI workflows for the `itsdarklikehell` fleet.

Single source of truth for lint / typecheck / test / build across all repos. Each repository gets a *thin* caller (`.github/workflows/ci.yml`) that `uses:` one of these workflows — so fleet-wide CI changes happen here, once.

## Development Visualization

<video src="https://raw.githubusercontent.com/itsdarklikehell/ci-templates/main/gource.mp4" controls width="100%"></video>

*Gource visualization showing the repository's commit history. See the [Gource workflow](.github/workflows/gource.yml) for details.*

<img src="https://img.shields.io/github/stars/itsdarklikehell/ci-templates?style=flat-square&color=blue" alt="Stars">
<img src="https://img.shields.io/github/forks/itsdarklikehell/ci-templates?style=flat-square&color=green" alt="Forks">
<img src="https://img.shields.io/github/license/itsdarklikehell/ci-templates?style=flat-square" alt="License">
<img src="https://img.shields.io/github/actions/workflow/status/itsdarklikehell/ci-templates/ci.yml?branch=main&label=CI&style=flat-square" alt="CI Status">

## Available Workflows

| Workflow | Use for | Language |
|----------|---------|----------|
| [`ci-node.yml`](.github/workflows/ci-node.yml) | Node.js / TypeScript projects | JavaScript, TypeScript |
| [`ci-python.yml`](.github/workflows/ci-python.yml) | Python projects | Python |
| [`ci-go.yml`](.github/workflows/ci-go.yml) | Go projects | Go |
| [`ci-generic.yml`](.github/workflows/ci-generic.yml) | Docker / Shell / other | Any |
| [`ci-shell.yml`](.github/workflows/ci-shell.yml) | Shell script projects | Bash, Shell |

## Quick Start

### Node.js / TypeScript

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: itsdarklikehell/ci-templates/.github/workflows/ci-node.yml@main
    secrets: inherit
```

### Python

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: itsdarklikehell/ci-templates/.github/workflows/ci-python.yml@main
    secrets: inherit
```

### Go

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: itsdarklikehell/ci-templates/.github/workflows/ci-go.yml@main
    secrets: inherit
```

### Docker / Shell / Generic

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: itsdarklikehell/ci-templates/.github/workflows/ci-generic.yml@main
    with:
      dockerfile: Dockerfile
      docker-build: true
      shellcheck: true
    secrets: inherit
```

### Shell scripts only

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: itsdarklikehell/ci-templates/.github/workflows/ci-shell.yml@main
    secrets: inherit
```

## Workflow Inputs

### ci-node.yml

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `node-version` | string | `"20"` | Node.js version or semver range |
| `package-manager` | string | `auto` | `auto` \| `npm` \| `yarn` \| `pnpm` \| `bun` |
| `lint-command` | string | `""` | Override lint command. `skip` disables. Empty = auto-detect `lint` script |
| `test-command` | string | `""` | Override test command. `skip` disables. Empty = auto-detect `test` script |
| `build-command` | string | `""` | Override build command. `skip` disables. Empty = auto-detect `build` script |
| `install-flags` | string | `""` | Extra flags appended to install command |
| `working-directory` | string | `"."` | Subdirectory containing `package.json` |

**Secrets:** `NPM_TOKEN` (optional) — for private registry auth.

### ci-python.yml

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `python-version` | string | `"3.12"` | Python version (semver) |
| `manager` | string | `auto` | `auto` \| `pip` \| `uv` \| `poetry` |
| `lint` | boolean | `true` | Run ruff when a ruff config is present |
| `typecheck` | boolean | `true` | Run mypy when a mypy config is present |
| `test` | boolean | `true` | Run pytest when tests exist |
| `build` | boolean | `false` | Build sdist/wheel with `python -m build` |
| `working-directory` | string | `"."` | Subdirectory containing `pyproject.toml` / `setup.py` |

**Secrets:** `CODECOV_TOKEN` (optional) — for Codecov uploads.

### ci-go.yml

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `go-version` | string | `"1.22"` | Go version (semver) |
| `lint` | boolean | `true` | Run gofmt / go vet / golangci-lint |
| `test` | boolean | `true` | Run `go test ./...` |
| `build` | boolean | `true` | Run `go build ./...` |

### ci-generic.yml

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `dockerfile` | string | `"Dockerfile"` | Path to Dockerfile (blank = skip) |
| `docker-build` | boolean | `true` | Build the Docker image (does not push) |
| `shellcheck` | boolean | `true` | Run shellcheck on `*.sh` files |

### ci-shell.yml

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `shellcheck` | boolean | `true` | Run shellcheck on `*.sh` files |
| `shfmt` | boolean | `true` | Run shfmt format check |

## How It Works

Each reusable workflow is designed to be **safe on first rollout**:

- **Node.js**: Only runs lint/test/build when the corresponding `package.json` script exists. The npm default stub test (`echo "Error: no test specified" && exit 1`) is treated as "no tests".
- **Python**: Only runs ruff/mypy when a config file (`ruff.toml`, `mypy.ini`, etc.) is present. This keeps first-run CI green for repos that haven't adopted those tools.
- **Go**: Lint (gofmt, go vet, golangci-lint) is advisory (`continue-on-error`) on first rollout. Test and build are hard gates.
- **Generic/Shell**: Shellcheck and hadolint are advisory. Docker build is a hard gate.

## Versioning

Workflows are referenced by branch (`@main`) or tag (`@v1`). For production use, pin to a specific tag:

```yaml
uses: itsdarklikehell/ci-templates/.github/workflows/ci-node.yml@v1
```

## Architecture

```
ci-templates/
├── .github/
│   ├── workflows/
│   │   ├── ci-node.yml      # Reusable Node.js CI
│   │   ├── ci-python.yml    # Reusable Python CI
│   │   ├── ci-go.yml        # Reusable Go CI
│   │   ├── ci-generic.yml   # Reusable Docker/Shell CI
│   │   ├── ci-shell.yml     # Reusable Shell CI
│   │   ├── ci.yml           # Self-test (validates all workflows)
│   │   ├── gource.yml       # Gource visualization
│   │   ├── labeler.yml      # PR labeler
│   │   ├── auto-assign.yml   # PR auto-assign
│   │   ├── release.yml      # Release automation
│   │   └── stale.yml        # Stale issue/PR management
│   ├── dependabot.yml       # Dependabot config
│   ├── renovate.json        # Renovate config
│   ├── labeler.yml          # Labeler config
│   └── CODEOWNERS           # Code owners
├── README.md
├── CONTRIBUTING.md
├── SECURITY.md
├── SUPPORT.md
└── CODE_OF_CONDUCT.md
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Security

See [SECURITY.md](SECURITY.md).

## Support

See [SUPPORT.md](SUPPORT.md).

## License

MIT — see [LICENSE](LICENSE) for details.
