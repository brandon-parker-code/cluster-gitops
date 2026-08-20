# Cluster GitOps

Flux config for the **shared AKS cluster**. App charts and values stay in per-app gitops repos.

Bootstrap (once, against this repo — not `email-consumer-service-gitops`):

```bash
az aks get-credentials --resource-group rg-ecs-prod --name aks-ecs-prod --overwrite-existing

export GITHUB_TOKEN='PAT with Contents: Read and write on cluster-gitops'

flux bootstrap github \
  --owner=brandon-parker-code \
  --repository=cluster-gitops \
  --branch=main \
  --path=clusters/prod \
  --personal
```

That creates `clusters/prod/flux-system/` in this repo. Do not add that folder by hand.

## App sources

[`clusters/prod/sources.yaml`](clusters/prod/sources.yaml) defines a GitRepository for [email-consumer-service-gitops](https://github.com/brandon-parker-code/email-consumer-service-gitops). Flux needs a **second** deploy key (or a PAT secret) for that repo; bootstrap’s deploy key only reads **this** repo.

```bash
# After bootstrap, create a deploy key on email-consumer-service-gitops and a matching secret:
flux create secret git email-consumer-service \
  --namespace=flux-system \
  --url=ssh://git@github.com/brandon-parker-code/email-consumer-service-gitops
# Follow the printed instructions to add the public key as a Deploy key (read-only) on that GitHub repo.
```

Then in **email-consumer-service-gitops**, point the HelmRelease chart `sourceRef` at GitRepository `email-consumer-service` (not `flux-system`). Until you do that, Flux will look for the Helm chart in **this** repo.

Add another app by adding another `GitRepository` + `Kustomization` under `clusters/prod/`.

Platform Terraform: [platform-terraform](https://github.com/brandon-parker-code/platform-terraform).
