# rpg-dice-mcp (deploy)

Kustomize manifests for the rpg-dice-mcp MCP server. Source code +
Dockerfile + CI live in
[dvystrcil/rpg-dice-mcp-docker](https://github.com/dvystrcil/rpg-dice-mcp-docker).

## What lives where

| Repo | Purpose |
|---|---|
| `dvystrcil/rpg-dice-mcp-docker` | Go source, Dockerfile, CI → publishes `harbor.sirddail.net/ai/rpg-dice-mcp:sha-<n>` |
| `dvystrcil/rpg-dice-mcp` (this repo) | Kustomize manifests applied to the cluster |
| `dvystrcil/argocd-projects/rpg-dice-mcp/` | Argo Application pointing at this repo's `base/` |
| `dvystrcil/argocd-image-updater/rpg-dice-mcp/` | ImageUpdater CR (per [`feedback_image_updater_crd_not_annotations`](https://github.com/dvystrcil/homelab/blob/main/memory/feedback_image_updater_crd_not_annotations.md) — no annotations) |
| `dvystrcil/gateway-services/charts/.../values.yaml` | HTTPRoute (per [`feedback_gateway_services_owns_routing`](https://github.com/dvystrcil/homelab) — chart-owned, not per-app) |

## Manifest set (`base/`)

- `namespace.yaml` — the `rpg-dice-mcp` namespace, labeled for the gateway-services chart selector
- `harbor-pull-secret.yaml` — `InfisicalSecret` projecting the Harbor pull dockerconfigjson
- `deployment.yaml` — single-replica Deployment; non-root; memory limit (per [`feedback_unbounded_pods_are_node_killers`](https://github.com/dvystrcil/homelab)); `/healthz` probes (NOT bare `/`, which the MCP streamable handler doesn't 200)
- `service.yaml` — ClusterIP on `:8080`
- `kustomization.yaml`

## Why split from `-docker`

Per the homelab `project_mcp_repo_split_pattern`: the build repo and the deploy repo are kept separate so:

- The image-updater watches Harbor → writes back to this repo (the canonical deploy state) without churning the build repo
- The Argo Application syncs from this repo on commit, not on every docker tag
- CI permissions stay narrow (build repo can push images; this repo can only commit manifests)

## Refs

- [dvystrcil/homelab#201](https://github.com/dvystrcil/homelab/issues/201) — scoping (TDD, ACs, three-repo rationale)
- [dvystrcil/rpg-dice-mcp-docker](https://github.com/dvystrcil/rpg-dice-mcp-docker)

## License

MIT — see `LICENSE`.
