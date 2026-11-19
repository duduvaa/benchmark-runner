# Windows VM Script Population (cnv-scenarios)

**Label:** `wayfinder:task`
**Status:** Resolved
**Blocked by:** [Script Integration Dispatch](019-script-integration-dispatch.md), [Phase 1 Closeout](030-phase1-closeout-multi-ns-and-flow2-restore.md)

## Question

Wire Windows VM population through the same `vm_population_method: script` path as RHEL, but invoke a different external script: `cnv-scenarios/run-workloads.sh hammerdb-mssql --os windows` (not `VMsPopulation.sh`, which is RHEL-only). Discriminate with `vm_population_script_args.os` (`windows` vs default `rhel`).

Resolve: Windows scenarios map `tests.yaml` args to script env vars, compute expected disk-util GB, run the script from `/home/kni/git/cnv-scenarios`, and treat stdout SUCCESS/FAILED (plus exit code) as pass/fail.

## Answer

- `vm_population_script_args.os`: `rhel` (default if omitted) | `windows`
- RHEL path unchanged (`VMsPopulation.sh`)
- Windows path: `cd /home/kni/git/cnv-scenarios && <env> ./run-workloads.sh hammerdb-mssql --os windows`
- Mapping: `windowsImageUrl`, `namespaces_to_backup`→`testNamespace`, `osdisksize`→`rootStorage`, `datadisksize`→`diskSize`, `vms`→`vmCount`, `data_disks`→`dataDisks`, optional gen params → `dirCount`/`filesPerDir`/`fileSize`
- Defaults: `data_disks=1`, `osdisksize=80Gi`, `datadisksize=150Gi`, gen `5/10/1G`
- Capacity: `data_disks==1` → `expectedTotalDiskUtilGB=48`; `>1` → extra=`(n-1)*dir*files*sizeGB`, total=`48+extra`
- **`ns` > 1:** supported. Same convention as RHEL (`namespaces_to_backup: <base>-ns1` + `ns: N`). Because `run-workloads.sh` takes one `testNamespace`, framework runs the **same** command once per sibling with only the namespace name changed (cnv-scenarios unchanged). Example TC 13.0.4: `ns: 3` → `perf-win2022-1vm-1os-1data-rbd-ns1`/`-ns2`/`-ns3`.
- **CBT (option A):** `cbt: true` allowed for Windows. After each successful `run-workloads.sh`, call `VMsEnableCBT.sh -ns <that sibling>`, wait for VMIs Running, then `VMsCBTstate.sh` and require state+label enabled. Flow2 backup resource-policy applies via existing `cbt: true` wiring; restore stays without resource policy.
- `increment_level` still rejected for Windows.
- **Data verify:** skip `VMsAddDataVerify.sh -verify-only` for Windows (RHEL SSH/`/home/data` baselines). Existence checks (VM count, VMI Running) still run; disk data is covered by `run-workloads.sh` SUCCESS at population time.
- Profile stub: `win2022-hammerdb-mssql.yaml` (script owns the real image URL)

### tests.yaml scenario to add in mpqe-scale-scripts

```yaml
- name: backup-dm-win2022-1vm-1os-1data-cephrbd
  testcase: 13.0
  testtype: backup
  dataset:
    role: kubevirt
    vm_profile: win2022-hammerdb-mssql
    vms_per_namespace: 10
    sc: ocs-storagecluster-ceph-rbd
    pv_size: 60Gi
  args:
    plugin: kubevirt
    use_cli: true
    OADP_CR_TYPE: backup
    OADP_CR_NAME: backup-dm-win2022-1vm-1os-1data-cephrbd
    backup_name: backup-dm-win2022-1vm-1os-1data-cephrbd
    testcase_timeout: 79200
    namespaces_to_backup: perf-win2022-1vm-1os-1data-rbd-ns1
    vm_population_method: script
    vm_population_script_args:
      os: windows
      windowsImageUrl: http://f01-h08-000-1029u.rdu2.scalelab.redhat.com:9002/winmssql2022.qcow2
      osdisksize: 80Gi
      datadisksize: 150Gi
      vms: 1
      data_disks: 2
      datadircount: 5
      datafilesperdir: 10
      datafilesize: 1G
```

Local checkout of `mpqe-scale-scripts` was unavailable when this ticket landed — add the scenario via mpqe MR when the repo is available.
