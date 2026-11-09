# Architecture: decision status

The intended architecture is described in the [README](../../README.md). This page separates what has been decided from what still needs research. Decisions with lasting impact are recorded as ADRs in [`../decisions/`](../decisions/README.md).

## Approved

| Decision | Notes |
|---|---|
| Single-cluster scope. No management cluster, CAPZ or ASO | Supersedes an earlier multi-cluster design |
| Region `westeurope`, cost-conscious lab | |
| Phase 1: disposable private AKS with BYOCNI and upstream Cilium | Destroyed after findings are documented |
| Phase 2: a new AKS cluster with Azure CNI Powered by Cilium | Cilium managed by AKS |
| Terraform for Azure resources and remote state | |
| Argo CD installed by Terraform, then self-managed through GitOps | Handover procedure still to be designed |
| Gateway API for ingress, TLS at the Gateway | |
| OpenTelemetry Astronomy Shop as the workload | Imported later into `app/astronomy-shop/` |

## Needs research or validation

| Topic | Question |
|---|---|
| kube-proxy replacement on BYOCNI | Depends on the preview `KubeProxyConfigurationPreview` feature. See the [sandbox README](../../terraform/environments/cilium-sandbox/README.md) |
| Private API access | How the workstation reaches the sandbox's private API server |
| Native routing | What the Azure underlay allows |
| ACNS | Whether its observability and security features justify the cost on the platform cluster |
| Application Gateway for Containers | ALB Controller-managed or Terraform-managed |
| Messaging | Event Hubs Kafka endpoint compatibility and authentication for each Kafka client |
| Cache and database | Managed Redis and Flexible Server networking and authentication per client |
| Entra authentication | Which services' languages, SDKs and drivers support it |
| Observability backends | In-cluster vs Azure Monitor |
| Flag state on Kubernetes | Writable shared volume vs Git-managed |

The workload-side questions are analysed in the [service map's migration decisions](../astronomy-shop/service-map.md#migration-decisions).
