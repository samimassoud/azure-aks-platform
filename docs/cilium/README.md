# Cilium: Phase 1 learning environment

Experiments and findings from the disposable BYOCNI sandbox. Nothing has been run yet.

## Scope

The sandbox runs **upstream open-source Cilium**, installed and configured directly. Hubble, Cilium's flow observability layer, is part of upstream Cilium and is enabled through Cilium's own configuration. It doesn't involve Azure's ACNS add-on. ACNS is relevant only to the AKS-managed Cilium of the Phase 2 platform cluster.

## Learning objectives

- [ ] kube-proxy replacement. This depends on a preview AKS feature; see the [sandbox README](../../terraform/environments/cilium-sandbox/README.md) for the gate and fallbacks.
- [ ] eBPF service load balancing and BPF map inspection
- [ ] Hubble flow observability
- [ ] Same-node and cross-node pod communication paths
- [ ] VXLAN and Geneve tunnelling
- [ ] Identity-based network policies
- [ ] Native routing, as a separate experiment within the Azure underlay's constraints
- [ ] Workstation access to the private AKS API server

## Evidence standard

Each experiment write-up records:

- **Versions:** AKS Kubernetes version, node image, Cilium version and its Helm values or CLI flags
- **Goal:** what was being tested and the expected behaviour, with references to Cilium or Azure documentation
- **Method:** the exact commands run
- **Observations:** relevant output (Cilium CLI, Hubble, `bpftool`, packet captures), trimmed but not edited
- **Conclusion:** what was confirmed, what was not, and open questions

Unverified expectations are labelled as such. No performance claims are made without a documented measurement.
