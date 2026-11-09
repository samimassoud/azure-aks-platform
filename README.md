# Azure Cloud-Native Platform on AKS

A cloud-native platform engineering project exploring Cilium and eBPF networking, Terraform, GitOps with Argo CD, Azure Workload Identity, managed Azure services, and OpenTelemetry.

> **Status: early stage.** This repository currently contains the project structure and an analysis of the workload application. No Azure infrastructure, AKS cluster, Cilium installation, Argo CD instance or application deployment exists yet. Everything under *Architecture* describes the intended design, which will change as each part is researched and validated. This is a learning and portfolio project, not a production-ready platform.

## Goals

- **Understand Cilium and eBPF networking in depth** by building, observing and breaking a self-managed installation before relying on a managed one.
- **Provision Azure infrastructure reproducibly** with Terraform and remote state.
- **Draw a clear ownership boundary** between Terraform (Azure resources) and Argo CD (everything inside the cluster, including Argo CD itself).
- **Prefer identity over secrets.** Use Microsoft Entra Workload ID for workload-to-Azure authentication wherever the specific language, SDK and driver support it.
- **Run a realistic, polyglot workload** whose telemetry shows whether the platform is working.
- **Back every claim with evidence.** Document the commands, outputs and versions that support each finding.
- **Keep costs low.** Run in a single region with disposable environments, and tear them down when they're idle.

## Architecture (intended)

Target region: **West Europe** (`westeurope`). One cluster at a time; there is no management cluster.

### Phase 1: disposable Cilium learning environment

A private AKS cluster created with **Bring Your Own CNI** (no network plugin preinstalled), running **self-managed upstream Cilium**. Learning objectives:

- kube-proxy replacement, eBPF service load balancing and BPF maps
- Hubble flow observability (part of upstream Cilium)
- same-node and cross-node pod communication
- VXLAN and Geneve tunnelling
- identity-based network policies
- native routing as a separate experiment, within the constraints of the Azure network underlay
- workstation access to the private AKS API server (method to be decided)

Known constraints:

- Microsoft support doesn't cover CNI-related issues on BYOCNI clusters.
- Disabling AKS-managed kube-proxy is currently a **preview** feature.

The environment is destroyed once its findings are documented in [`docs/cilium/`](docs/cilium/).

### Phase 2: platform cluster

A **new** AKS cluster using **Azure CNI Powered by Cilium**, where AKS installs and manages the Cilium lifecycle. Nothing from the sandbox carries over except documentation.

Some capabilities on managed Cilium (container network observability, FQDN filtering, L7 policies) come from **Advanced Container Networking Services (ACNS)**, a separately billed AKS feature. Whether to enable it is an open cost decision. The Phase 1 sandbox doesn't depend on ACNS.

### Platform capabilities (planned)

| Area | Intended approach |
|---|---|
| Infrastructure | Terraform with remote state |
| GitOps | Argo CD, installed initially by Terraform, then handed over to manage itself from Git |
| Ingress and TLS | Gateway API on Application Gateway for Containers (ALB Controller), TLS at the Gateway, cert-manager |
| DNS | ExternalDNS with Azure DNS |
| Managed data services | Azure Managed Redis, Azure Event Hubs (Kafka endpoint), Azure Database for PostgreSQL Flexible Server |
| Workload identity | Microsoft Entra Workload ID |
| Network security | Cilium network policies; Hubble |
| Observability | OpenTelemetry, starting from the application's existing instrumentation |
| Optional | Service-mesh experiments with Cilium |

### Ownership boundary

| Owned by Terraform | Owned by Argo CD (after handover) |
|---|---|
| Resource groups, networking, AKS | All in-cluster configuration and add-ons |
| Managed identities and federated credentials | Argo CD itself |
| Managed data services, DNS zones | The application workload |
| Remote state; the initial Argo CD install | |

Where ambiguous resources such as the Application Gateway for Containers instance belong is still an open question.

## Roadmap

| Step | Status |
|---|---|
| Workload analysis (service map and dependency inventory) | Documented |
| Repository scaffold | In progress |
| Terraform remote-state bootstrap | Planned |
| Phase 1: BYOCNI sandbox and Cilium experiments | Planned |
| Phase 2: platform cluster on Azure CNI Powered by Cilium | Planned |
| Argo CD bootstrap and self-management handover | Planned |
| Ingress, TLS and DNS | Planned |
| Managed Azure dependencies and Workload ID | Planned |
| Application deployment | Planned |
| Network policies and observability | Planned |

## Workload: OpenTelemetry Astronomy Shop

The workload is the [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo) ("Astronomy Shop"): a polyglot microservice web shop, instrumented with OpenTelemetry, with built-in load generation and feature-flag-driven failure scenarios. It's developed by the OpenTelemetry community. This project uses it as a workload and doesn't claim authorship.

Its source will be imported into `app/astronomy-shop/` with its upstream provenance recorded. Over time, some in-cluster dependencies are planned to move to managed Azure services:

- Valkey → Azure Managed Redis
- Apache Kafka → Azure Event Hubs, after validating Kafka client authentication and protocol compatibility
- the local PostgreSQL container → Azure Database for PostgreSQL Flexible Server
- password and connection-string authentication → Microsoft Entra identity, where each client library supports it

Each change will be driven by the service source code and tests. None of it has been implemented yet.

- [Overview](docs/astronomy-shop/overview.md): what the shop does and why it suits this platform
- [Service map](docs/astronomy-shop/service-map.md): every component, its interfaces, dependencies and configuration, plus the open migration decisions

## Repository layout

```text
.github/workflows/   CI workflows (planned)
app/                 Application source; the Astronomy Shop will live in app/astronomy-shop/
argocd/              Argo CD bootstrap, root application and application definitions (planned)
terraform/           Remote-state bootstrap, reusable modules and per-environment configurations
  environments/
    cilium-sandbox/  Phase 1 disposable cluster
    platform/        Phase 2 platform cluster
docs/
  architecture/      Architecture overview and decision status
  decisions/         Architecture decision records
  cilium/            Phase 1 experiments and findings
  astronomy-shop/    Workload analysis
scripts/             Helper scripts (planned)
```

Directories marked *planned* appear when they get their first content.

## Cost approach

- The Phase 1 cluster is disposable, and environments are destroyed when idle.
- SKUs and tiers are chosen for a learning lab, and each choice is documented with its trade-offs.
- No cost figures are claimed here. Pricing is checked against Azure's own pricing pages when each service is introduced.

## Licence

Unless otherwise indicated, original project materials are licensed under the [Apache License 2.0](LICENSE). The Astronomy Shop application retains its upstream licensing and notices.
