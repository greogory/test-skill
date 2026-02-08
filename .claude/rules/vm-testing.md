# Phase V: VM Testing Configuration

## Default Test VM

**test-vm-cachyos**: CachyOS with KDE desktop, 4GB RAM, 4 vCPUs, 40GB disk
- Location: `/var/lib/libvirt/images/test-vm-cachyos.qcow2`
- ISO Location: `/hddRaid1/ISOs/`
- Auto-Detection: Phase V automatically finds VMs with "test" or "dev" in the name

## Manifest Template

`templates/vm-test-manifest.json` — copy to project root to customize VM testing.

## VM Management

```bash
sudo virsh list --all
sudo virsh start test-vm-cachyos
virt-viewer test-vm-cachyos
sudo virsh snapshot-create-as test-vm-cachyos clean-install --description "Fresh install"
sudo virsh snapshot-revert test-vm-cachyos clean-install
```

## When Phase V Runs

- Project has `vm-test-manifest.json` with `"enabled": true`
- Project has dangerous operations (PAM, systemd, kernel)
- User explicitly requests `--phase=V`
- Phase 0/1 detects install scripts modifying system-level configs

## VM Snapshot Workflow (MANDATORY)

All VM-isolated tests MUST follow this snapshot lifecycle:

```bash
# 1. BEFORE tests: Create pre-test snapshot
sudo virsh snapshot-create-as test-vm-cachyos pre-test-$(date +%Y%m%d-%H%M%S) \
    --description "Pre-test state before /test run"

# 2. RUN tests

# 3. AFTER tests: Revert to pre-test snapshot
sudo virsh snapshot-revert test-vm-cachyos <snapshot-name>

# 4. CLEANUP: Delete the temporary pre-test snapshot
sudo virsh snapshot-delete test-vm-cachyos <snapshot-name>
```

### Snapshot Types

| Snapshot | Purpose | Lifetime |
|----------|---------|----------|
| `clean-install` | Permanent baseline (fresh OS + SSH) | Permanent |
| `pre-test-YYYYMMDD-HHMMSS` | State before specific test run | Deleted after test |
