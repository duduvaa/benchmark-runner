# Multi-Namespace VM Scenarios

**Label:** `wayfinder:task`  
**Status:** Open  
**Blocked by:** [Script Integration Dispatch](019-script-integration-dispatch.md)

## Question

Extend VM dataset creation to support `total_namespaces > 1` with user-specified VMs per namespace. The pod flow already supports multi-NS; mirror the pattern for VMs.

**Script-based path:** VMsPopulation.sh already supports `-ns N` to create multiple namespaces. Ticket 018 adds `-namespace` to control the base name. The script appends `-ns${i}` in its loop, so `-namespace kubevirt-test -ns 3` would create `kubevirt-test-ns1`, `kubevirt-test-ns2`, `kubevirt-test-ns3`. The `namespaces_to_backup` in scenario args needs to list all namespaces (comma-separated or label-selector).

**Native Python path:** `create_vm_dataset()` currently handles a single namespace. Extend to iterate over `total_namespaces`:

1. Iterate namespace creation for `total_namespaces`
2. Per-namespace VM creation with batching
3. Per-namespace validation (pre-backup and post-restore)
4. Per-namespace cleanup (stop VMs in ALL namespaces before deleting ANY — order matters for finalizers)
5. `namespaces_to_backup` in scenario args needs to list all namespaces (comma-separated or label-selector)

The script-based path (Phase 1) gets multi-NS nearly for free via `-ns N`. The native path (Phase 2) needs the iteration logic implemented.

Resolve: both script-based and native multi-NS paths work, unit tested, validated on cluster with 2+ namespaces.
