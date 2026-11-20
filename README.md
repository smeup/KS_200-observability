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
dashboards/
├── ahd/         cluster baseline: argocd, infra, keycloak, node-exporter
├── databases/   mongodb, postgresql
└── grain/       platform services: as400proxy, restapi, services
```

Each subfolder becomes a Grafana folder. Every subfolder must contain a `_folder.json` with a stable folder UID
(`metadata.name`, never change it) and the folder title shown in Grafana (`spec.title`). Adding, moving or renaming dashboards needs no change in the `commons` chart.

Alert rules still live in the `commons` chart of [smeup/helm-charts](https://github.com/smeup/helm-charts)
(`charts/commons/alerts/`), because Git Sync supports only dashboards and folders for now.
They will move to an `alerts/` folder here once Grafana supports them.

## Dashboard formats

Both formats accepted by Grafana are in use:

| Format | Identifier | Files |
|---|---|---|
| v2 resource (`apiVersion: dashboard.grafana.app/v2`) | `metadata.name` | argocd, as400proxy, infra, restapi, services |
| classic JSON | `uid` | keycloak, mongodb, node-exporter, postgresql |

The identifier must be unique and must never change: Grafana uses it to match the dashboard across updates.

## How Grafana consumes this repository

Grafana pulls `dashboards/` from `main` with **Git Sync** (Grafana 13+), configured by the `commons` chart (`grafana.gitSync`):
a read-only `Repository` (`type: git`, `workflows: []`, `target: folderless`).
The repository is public, so no token is needed.

- Grafana pulls every 24 hours (`grafana.gitSync.intervalSeconds`).
- To get a merge on `main` immediately: **Administration → Provisioning → Repositories → Platform dashboards → Pull**.
- Dashboards synced from Git cannot be saved from the Grafana UI: change them here.

## Changing a dashboard

1. Edit the dashboard in Grafana (as a copy, or on a test instance).
2. Export it: **Share → Export → Save to file**, with *Export for sharing externally* **off**
   (keeps datasource UIDs, no `${DS_...}` inputs).
3. Replace the file under `dashboards/` and open a pull request to `develop`.

Merged changes reach Grafana at the next pull (see above).

CI (`.github/workflows/validate.yaml`) checks that every dashboard is valid JSON with a unique identifier.
