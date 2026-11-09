# Argo CD

GitOps configuration for the platform cluster. Nothing is implemented yet.

## Intended layout

| Directory | Purpose |
|---|---|
| `bootstrap/` | What Argo CD needs to start managing itself: its own installation values and the root application |
| `root/` | The root ("app of apps") definition that points Argo CD at `apps/` |
| `apps/` | One Argo CD Application per platform add-on and per workload |

These directories are created with their first content.

## Terraform → Argo CD handover

1. Terraform installs Argo CD once, so a GitOps controller exists in the cluster.
2. Argo CD is pointed at this repository and takes over its own configuration from `bootstrap/`.
3. From then on, in-cluster state (Argo CD itself, add-ons and the application) changes only through Git.

The exact handover procedure is to be designed and documented before it's carried out, so Terraform and Argo CD never both manage the same objects.
