# CLAUDE.md: project guide

This file is the authoritative guide for working in this repository. Where it conflicts with older notes, this file wins.

## 1. What the project is

**Azure Cloud-Native Platform on AKS** is a portfolio and learning project. It explores Cilium and eBPF networking, Terraform, GitOps with Argo CD, Microsoft Entra Workload ID, managed Azure data services and OpenTelemetry.

- Region: **West Europe (`westeurope`)**.
- It's a cost-conscious lab, not a production platform.
- Nothing has been deployed yet. See `README.md` → Roadmap for current status.

## 2. Scope

**In scope:**

- A single AKS cluster at a time, in two phases (§3).
- Terraform-managed Azure infrastructure with remote state.
- Argo CD installed by Terraform, then self-managed through GitOps.
- cert-manager, Application Gateway for Containers (ALB Controller) and Gateway API, ExternalDNS with Azure DNS.
- Azure Managed Redis, Azure Event Hubs, Azure Database for PostgreSQL Flexible Server.
- Microsoft Entra Workload ID.
- The OpenTelemetry Astronomy Shop as the workload.
- Cilium network policies and Hubble. Service-mesh experiments are optional.

**Out of scope:**

- **CAPZ, ASO and a separate management cluster.**
- Multiple environments, multiple regions, multiple tenants or subscriptions.

Don't reintroduce out-of-scope items, even if older notes mention them, unless the owner explicitly revisits that decision.

## 3. The two phases are separate clusters

| | Phase 1: Cilium sandbox | Phase 2: platform cluster |
|---|---|---|
| Purpose | Learn Cilium/eBPF internals | Run the platform and workload |
| AKS networking | BYOCNI (`--network-plugin none`) | Azure CNI Powered by Cilium |
| Who manages Cilium | The owner (upstream Cilium) | AKS |
| Hubble | Upstream Hubble, configured through Cilium itself; no ACNS involved | AKS observability features come through ACNS (separately billed; open decision) |
| Lifetime | Disposable: destroyed once findings are documented in `docs/cilium/` | Long-lived for the duration of the project |
| Terraform | `terraform/environments/cilium-sandbox/` | `terraform/environments/platform/` |

- Nothing carries over from the sandbox to the platform cluster except documentation.
- Don't assume a capability observed in the sandbox exists on AKS-managed Cilium, or the other way round. Check each against the relevant documentation.
- The sandbox's kube-proxy replacement depends on a **preview** AKS feature (`KubeProxyConfigurationPreview`). See `terraform/environments/cilium-sandbox/README.md`.

## 4. Ownership: Terraform vs Argo CD

- **Terraform owns Azure resources**:
  - resource groups, networking, AKS
  - managed identities and federated credentials
  - managed data services, DNS zones
  - remote state
  - the *initial* Argo CD installation
- **Argo CD owns in-cluster state** after the handover, including its own configuration, add-ons and the application.
- After the handover, Terraform must not keep managing what Argo CD manages. The handover procedure will be designed and documented before it's carried out.
- **Open:** who owns the Application Gateway for Containers resource (ALB Controller-managed vs Terraform-managed).

## 5. Decision status

- Approved decisions and open questions are tracked in `docs/architecture/README.md`. ADRs go in `docs/decisions/`.
- Treat anything not listed as approved as a proposal that needs research and the owner's review.

## 6. The Astronomy Shop

- **Location:**
  - Today the source is a separate clone at `opentelemetry-demo/`. It's excluded from this repository's Git index and has its own Git metadata, which must not be modified.
  - It will be imported into `app/astronomy-shop/` in a dedicated, reviewed task. The planned approach is `git subtree --squash`.
- **Service documentation:**
  - `docs/astronomy-shop/overview.md` and `docs/astronomy-shop/service-map.md` were derived from the source at a pinned commit.
  - Update them **from the source code**, never from assumptions about the target architecture. Keep the pinned commit current when re-verifying.
- **Upstream files:**
  - Never edit the application's own `CLAUDE.md` or `AGENTS.md`.
  - Never remove or alter its LICENSE, copyright or attribution notices.
- **Dependency replacements** are planned, not implemented:
  - Valkey → Azure Managed Redis
  - Apache Kafka → Azure Event Hubs, only after validating each Kafka client's authentication and protocol compatibility
  - PostgreSQL container → Flexible Server
  - passwords → Entra authentication, only where the specific language, SDK and driver support it
  - Each change must be guided by the service's actual source code and tests. The service map lists the current client libraries and configuration.
- **Upstream `.env` files:** upstream tracks `.env`, `.env.override` and `src/react-native-app/.env`. Before import, inspect them, sanitise any credentials, keep example templates and run a Gitleaks scan.

## 7. Engineering principles

- **Least privilege:** narrowly scoped identities and role assignments; no shared credentials.
- **Cost awareness:** pick lab-appropriate SKUs, destroy idle environments, and state the cost implications of a proposal.
- **Reproducibility:** everything is defined in code or documented as a repeatable procedure.
- **Incremental implementation:** make small, reviewable changes, each with a clear purpose.
- **Documentation and evidence:** findings record the commands, outputs and versions that support them. Never claim results, benchmarks or improvements that weren't measured.

## 8. Working rules

- **Inspect before proposing.**
  - Read the existing code and docs in this repo first.
  - Check official documentation: Microsoft Learn (use the Microsoft Learn MCP tools) for Azure, the Cilium docs for Cilium, and upstream docs for other tools.
- **Don't invent.**
  - No made-up service dependencies, configuration variables, Azure SDK or driver support, Azure feature availability, or Cilium capabilities.
  - If something isn't in the code or docs, write **"not found"**. Mark unverified claims as such.
- **Distinguish** what is implemented, approved, proposed and still under research.
- **Stop and ask for review before:**
  - any infrastructure change (`terraform apply`/`destroy`, `az`/`kubectl`/`helm` commands that mutate state)
  - Git commits or pushes, or changes to remotes
  - substantial architectural choices, or adding a new component or dependency
- **Never commit** secrets, `.env` files with real values, Terraform state, `*.tfvars` with real values, kubeconfigs, certificates or private keys. Use `git add` with explicit paths, never `git add .`, and run Gitleaks before pushing.
- **Ingress** uses Gateway API (Gateway + HTTPRoutes), not `Ingress`. TLS terminates at the Gateway.
- **Docs** are Markdown and live under `docs/`.

## 9. Where to look

| Path | Contents |
|---|---|
| `README.md` | Project overview, intended architecture, roadmap |
| `docs/architecture/` | Decision status (approved vs open) |
| `docs/decisions/` | Architecture decision records |
| `docs/cilium/` | Phase 1 objectives, experiments and findings |
| `docs/astronomy-shop/` | Workload overview and service map |
| `terraform/` | Terraform layout and environment scopes |
| `argocd/` | GitOps layout and handover model |
| `app/` | Application source (after import) |
