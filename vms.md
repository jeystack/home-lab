# VM Inventory – KVM Home Lab

Host: Ubuntu 26.04.1 LTS on ThinkPad T14 G2 · Hypervisor: KVM/QEMU + libvirt (virt-manager)
Why the lab is isolated: see [README.md](README.md#why-the-lab-is-isolated).

> **Decommissioned 2026-10-17** together with the dual boot – all VMs below were deleted with the Ubuntu partition. Kept as a reference for the rebuild on Proxmox.

> Credentials are **not** stored here – they live in `private/` (excluded via .gitignore).

## Networks

| Network | Type | Subnet | Internet | Used for |
|---------|------|--------|----------|----------|
| default | NAT | 192.168.122.0/24 | yes | initial setup & updates only |
| lab-isolated | isolated (no forwarding) | 192.168.100.0/24 | no | all lab work |

---

## kali – Attacker VM

| Field | Value |
|-------|-------|
| Purpose | Attacker machine: scanning & enumeration |
| OS / Version | Kali Linux 2026.2 |
| Source | kali.org – QEMU image |
| SHA256 verified | yes - 2026-10-15 |
| vCPU / RAM / Disk | 2 / 4 GB / 80 GB (qcow2, thin provisioned) |
| Network | default (updates) → lab-isolated |
| IP | NAT 192.168.122.8 (never moved to lab-isolated) |
| Snapshots | `clean-updated` – 2026-10-15 |
| Status | decommissioned 2026-10-17 (was: running on NAT, fully upgraded – 2026-10-15) |

**Notes / Issues**
- Imported the official QEMU image with `virt-install --import` instead of installing from ISO
- Default password changed right after first login
- During `full-upgrade`, libc6 asked to restart services automatically → answered yes (lab VM, nothing critical running)
- Display artifacts during the upgrade (graphics stack replaced while running) → fixed by a reboot

---

## ubuntu-srv (target)

| | |
|---|---|
| OS | Ubuntu Server 26.04.1 LTS ("resolute") |
| vCPU / RAM / Disk | 2 / 2 GB / 25 GB (qcow2, LVM, no encryption) |
| IP | NAT 192.168.122.149 (never moved to lab-isolated) |
| Snapshots | `clean-install` – 2026-10-15 |
| Status | decommissioned 2026-10-17 (was: running on NAT, fully upgraded – 2026-10-15) |

**Notes / Issues**
- Installed in the NAT network first so the installer could reach package mirrors and pull security updates (confirmed in the installer log). `lab-isolated` has no forwarding and no DNS by design. Acceptable risk: official image, SHA256-verified, not intentionally vulnerable. Vulnerable machines (VulnHub) go straight into `lab-isolated`.
- OS variant `ubuntu25.10` – osinfo-db on the host doesn't know 26.04 yet; closest available
- 2 GB RAM is below libvirt's recommendation (3 GB) for that variant; fine for a headless server, increase if needed
- Guided LVM leaves free space in the volume group on purpose → extend later with `lvextend` when needed
- OpenSSH server installed, password auth still enabled → switch to key-only as part of hardening
- Ubuntu Pro skipped
- Rebooted after `full-upgrade` before the snapshot (needrestart reported deferred services)
- Timezone left at UTC (server default; keeps logs comparable)

---

## vulnhub-01 – Vulnerable VM

| Field | Value |
|-------|-------|
| Purpose | Intentionally vulnerable target for scanning practice |
| Machine name | _TBD (name + VulnHub link)_ |
| Source format | _.ova → converted to .qcow2 with `qemu-img convert`_ |
| SHA256 / MD5 verified | _yes/no – date_ |
| vCPU / RAM / Disk | _per VulnHub description_ |
| Network | **lab-isolated only – never NAT** |
| IP (lab-isolated) | _TBD (found via scan)_ |
| Snapshots | _`imported` – date_ |
| Status | _planned for the Proxmox lab_ |

**Notes / Issues**
- _..._
