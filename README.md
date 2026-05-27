# maxoperf-helm-charts

GitOps companion repository for **Maxoperf** Kubernetes delivery. Holds **ECR image manifests** and Helm image overlays that Argo CD consumes when deploying to EKS.

## Layout (mirrors `maxoperf`)

| Path | Purpose |
| --- | --- |
| `deploy/ecr-image-versions.yaml` | SSOT for published ECR tags per component (`schemaVersion: 1`) |
| `deploy/helm/maxoperf/values-staging-images.yaml` | Generated staging image overlay for Helm |
| `deploy/helm/maxoperf/values-prod-images.yaml` | Generated production image overlay for Helm |

Files under `deploy/helm/maxoperf/values-*-images.yaml` are **generated** by `scripts/sync-ecr-tags-to-env-values.mjs` in the [maxoperf](https://github.com/speedy-develop/maxoperf) repo from `deploy/ecr-image-versions.yaml`.

## How manifests are updated

On every **push to `main`** in `maxoperf`, the CI workflow:

1. Runs the full affected test + Docker build matrix.
2. Pushes changed images to ECR.
3. Updates this repo’s manifest files on `main` (commit message `chore(ecr): bump image overlays …`).

Pull requests in `maxoperf` still commit image overlays to the **PR branch** in `maxoperf` for review; only **`main`** promotions land here.

## Deploying to EKS

Use **Deploy EKS with Argo CD** in either repository:

- **`maxoperf`** — full deploy workflow (Helm charts + scripts live there today).
- Choose **images source repo** (`maxoperf` or `maxoperf-helm-charts`) and **images source ref** (branch/tag/SHA) to pin which ECR tags are applied, independently of the Helm chart git revision.

### Secrets

`maxoperf` CI **must** have **`HELM_CHARTS_REPO_TOKEN`** — the default `GITHUB_TOKEN` in Actions is scoped only to `maxoperf` and cannot push here. One-time setup from a machine with `gh auth login`:

```bash
bash scripts/github-setup-helm-charts-publish-token.sh
```

(in the **maxoperf** repo). That stores a PAT with `contents: write` on this repo and triggers **`repository_dispatch`**; this repo’s workflow commits using its own `GITHUB_TOKEN`.

## Related

- Application charts: [speedy-develop/maxoperf](https://github.com/speedy-develop/maxoperf) (`deploy/helm/maxoperf/`)
- Argo CD examples: `maxoperf/deploy/gitops/argocd/`
