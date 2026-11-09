# Terraform

Azure infrastructure for the project. Nothing is implemented yet.

## Intended layout

| Directory | Purpose |
|---|---|
| `bootstrap/` | The remote-state backend (storage for Terraform state), created first |
| `modules/` | Reusable modules shared by the environments |
| `environments/cilium-sandbox/` | Phase 1: disposable private AKS cluster with BYOCNI ([README](environments/cilium-sandbox/README.md)) |
| `environments/platform/` | Phase 2: the platform cluster on Azure CNI Powered by Cilium ([README](environments/platform/README.md)) |

## Ownership

- **Terraform owns Azure resources:**
  - resource groups, networking, AKS
  - managed identities and federated credentials
  - managed data services, DNS zones
  - the initial Argo CD installation
- **In-cluster state belongs to Argo CD** after the handover. See [`argocd/`](../argocd/README.md).

## Conventions

- State is never committed. `.terraform.lock.hcl` files are committed.
- Real `*.tfvars` values are never committed. Sanitised `*.tfvars.example` files may be.
- No `terraform apply` or `destroy` without a reviewed plan.
