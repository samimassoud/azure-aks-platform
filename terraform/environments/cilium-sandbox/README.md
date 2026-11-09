# Phase 1: Cilium sandbox (disposable)

A private AKS cluster created with **Bring Your Own CNI** (`--network-plugin none`) and **self-managed upstream Cilium**. It exists for learning and is destroyed once its findings are documented in [`docs/cilium/`](../../../docs/cilium/README.md). Nothing is implemented yet.

## Known constraints (Microsoft Learn)

- **Support boundary.** On BYOCNI clusters, Microsoft support doesn't cover CNI-related issues, including most pod-to-pod traffic. Non-CNI AKS issues remain supported. Cilium problems here are self-supported, using Cilium's documentation and community.
- **A pod CIDR must be specified** at creation, outside Azure's reserved ranges.
- **Disabling the AKS-managed kube-proxy is a preview feature.**
  - It uses the kube-proxy configuration `enabled: false` and needs the `KubeProxyConfigurationPreview` feature registered on `Microsoft.ContainerService`, with the provider re-registered afterwards.
  - It also needs the `aks-preview` CLI extension, or ARM API version `2022-08-02-preview` or later.
  - Preview features have no SLA and only best-effort support. That's acceptable here only because the cluster is disposable.

## Gate before the kube-proxy replacement experiment

- [ ] `KubeProxyConfigurationPreview` reaches *Registered* on the lab subscription.
- [ ] The feature and the chosen VM sizes work in `westeurope`. The documentation lists no region limits; this is checked, not assumed.
- [ ] Pick an AKS Kubernetes version supported in `westeurope`, and a Cilium release whose documentation lists that Kubernetes version and the node OS/kernel as compatible. Record all versions.
- [ ] Find out whether the Terraform `azurerm` provider exposes the kube-proxy configuration. Not yet verified. If it doesn't, evaluate `azapi` against the preview API or a documented manual step.

**If the preview blocks the experiment**, choose between these fallbacks. Each must be validated before use:

1. Keep AKS-managed kube-proxy and do the other objectives. Check Cilium's documentation for whether and how its kube-proxy replacement can coexist with an existing kube-proxy.
2. Run the kube-proxy replacement experiment on a separate, short-lived self-managed cluster.
3. Record the limitation as a finding.

## Other open items

- How the workstation reaches the private API server. Method not yet chosen.
- Native routing on the Azure underlay: what the VNet allows for pod CIDR routes. To be researched.
