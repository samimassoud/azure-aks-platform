# Phase 2: platform cluster

A new AKS cluster using **Azure CNI Powered by Cilium**, where AKS installs and manages Cilium. It runs the platform add-ons and the Astronomy Shop. Nothing is implemented yet.

## Known facts (Microsoft Learn)

- Clusters on Azure CNI Powered by Cilium don't use kube-proxy.
- L3/L4 `CiliumNetworkPolicy` and `CiliumClusterwideNetworkPolicy` are supported.
- Container network observability, FQDN filtering and L7 policies come through **Advanced Container Networking Services (ACNS)**, a separately billed feature.
- Application Gateway for Containers supports Azure CNI and Azure CNI Overlay with Cilium network policies. It must be in the cluster's VNet, in a dedicated `/24` subnet, and the cluster needs Workload Identity enabled.

## Open items

- Whether to enable ACNS, and which parts, given its cost.
- IP address management mode: overlay or pod subnet.
- Who owns the Application Gateway for Containers resource: ALB Controller or Terraform.
- Network layout for the managed data services (private endpoints vs VNet integration). See the [service map's migration decisions](../../../docs/astronomy-shop/service-map.md#migration-decisions).
