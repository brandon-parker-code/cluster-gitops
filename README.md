# Cluster GitOps

Flux config for each AKS cluster. App charts and values stay in per-app gitops repos.

| Path | Cluster (Terraform `environment`) |
| --- | --- |
| [`clusters/prod/`](clusters/prod/) | `aks-ecs-prod` |
| [`clusters/dev/`](clusters/dev/) | `aks-ecs-dev` |

**Flux is installed by Terraform** in [platform-terraform](https://github.com/brandon-parker-code/platform-terraform) (`azurerm_kubernetes_cluster_extension` `microsoft.flux` + `azurerm_kubernetes_flux_configuration` path `./clusters/<environment>`). Do **not** run `flux bootstrap`.

Set `github_flux_token` in `terraform.tfvars` to a PAT with **Contents: Read** on this repo and on [email-consumer-service-gitops](https://github.com/brandon-parker-code/email-consumer-service-gitops). Terraform uses it to clone this repo and to create the `email-consumer-service` git secret for the app GitRepository.

HelmReleases in the app gitops repo are applied by Flux’s **helm-controller** (part of the AKS Flux extension). There is no separate Helm install on the cluster.

Add another app by adding another `GitRepository` + `Kustomization` under `clusters/<env>/` (and a matching git secret in Terraform if the repo is private).
