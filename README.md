# ci-templates


## Development Visualization

<video src="https://raw.githubusercontent.com/itsdarklikehell/ci-templates/main/gource.mp4" controls width="100%"></video>

*Gource visualization showing the repository's commit history. See the [Gource workflow](.github/workflows/gource.yml) for details.*


<img src="https://img.shields.io/github/stars/itsdarklikehell/ci-templates?style=flat-square&color=blue" alt="Stars">
<img src="https://img.shields.io/github/forks/itsdarklikehell/ci-templates?style=flat-square&color=green" alt="Forks">
<img src="https://img.shields.io/github/license/itsdarklikehell/ci-templates?style=flat-square" alt="License">
<img src="https://img.shields.io/github/actions/workflow/status/itsdarklikehell/ci-templates/ci.yml?branch=main&label=CI&style=flat-square" alt="CI Status">

Reusable GitHub Actions CI workflows for the `itsdarklikehell` fleet.

Single source of truth for lint / typecheck / test / build across all repos. Each repository gets a *thin* caller (`.github/workflows/ci.yml`) that `uses:` one of these workflows — so fleet-wide CI changes happen here, once.

## Installatie

### Gebruik in een repo

Voeg een workflow-bestand toe aan je repo:

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: itsdarklikehell/ci-templates/.github/workflows/ci-node.yml@main
    secrets: inherit
```

### Beschikbare workflows

| Workflow | Use for |
|----------|---------|
| `ci-node.yml` | Node.js / TypeScript (`package.json`) |
| `ci-python.yml` | Python projects |
| `ci-shell.yml` | Shell scripts |

## Gebruik

Na het toevoegen van de workflow wordt CI automatisch uitgevoerd bij elke push en pull request.

```bash
# Test lokaal (waar beschikbaar)
npm test
# of
pytest
```

## Bijdragers

- [itsdarklikehell](https://github.com/itsdarklikehell) — Onderhouder

## Licentie

MIT — zie [LICENSE](LICENSE) voor details.
