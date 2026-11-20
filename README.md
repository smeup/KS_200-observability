# KS_200-observability

Grafana resources for the smeup platform, managed as code.
This repository is the **source of truth**: every Grafana instance that points here gets what is merged on `main`.

## Branches

| Branch | Purpose |
|---|---|
| `main` | What Grafana instances consume. Updated only by a PR `develop` → `main`. |
| `develop` | Default branch, integration. `feature/*`, `fix/*`, `chore/*` branch from it and merge back via PR. |

Changes flow one way only, `develop` → `main`: `main` is never edited directly and never merged back.
Merge the `develop` → `main` PR with **Create a merge commit**, not squash,
otherwise the next `develop` → `main` PR shows already released changes again.

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `chore:`, `docs:`).

## Layout

```
dashboards/   Grafana dashboards (JSON)
```

Alert rules still live in the `commons` chart of [smeup/helm-charts](https://github.com/smeup/helm-charts)
(`charts/commons/alerts/`), because Git Sync supports only dashboards and folders for now.
They will move to an `alerts/` folder here once Grafana supports them.

## Dashboard formats

Both formats accepted by Grafana are in use:

| Format | Identifier | Files |
|---|---|---|
| v2 resource (`apiVersion: dashboard.grafana.app/v2`) | `metadata.name` | argocd, as400proxy, infra, restapi, services |
| classic JSON | `uid` | keycloak, mongodb, postgresql |

The identifier must be unique and must never change: Grafana uses it to match the dashboard across updates.

## How Grafana consumes this repository

The repository is public, so no token is needed to read it.

- **Raw URL** (Grafana Helm chart `dashboards.<provider>.<name>.url`):
  `https://raw.githubusercontent.com/smeup/KS_200-observability/main/dashboards/<file>.json`.
  Dashboards are downloaded when the Grafana pod starts.
- **Git Sync** (Grafana 13+), read-only: a `Repository` resource with `workflows: []` and `path: dashboards/`.

## Changing a dashboard

1. Edit the dashboard in Grafana.
2. Export it: **Share → Export → Save to file**, with *Export for sharing externally* **off**
   (keeps datasource UIDs, no `${DS_...}` inputs).
3. Replace the file in `dashboards/` and open a pull request.

Changes made only in the Grafana UI are lost on the next sync or restart.

CI (`.github/workflows/validate.yaml`) checks that every dashboard is valid JSON with a unique identifier.
