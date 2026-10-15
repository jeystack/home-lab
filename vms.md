# VM Inventory – KVM Home Lab

Host: Ubuntu 26.04.1 LTS on ThinkPad T14 G2 · Hypervisor: KVM/QEMU + libvirt (virt-manager)
Why the lab is isolated: see [README.md](README.md#why-the-lab-is-isolated).

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
| IP (lab-isolated) | _TBD_ (currently NAT: 192.168.122.8) |
| Snapshots | `clean-updated` – 2026-10-15 |
| Status | running on NAT, fully upgraded - 2026-10-15 |

**Notes / Issues**
- Imported the official QEMU image with `virt-install --import` instead of installing from ISO
- Default password changed right after first login
- During `full-upgrade`, libc6 asked to restart services automatically → answered yes (lab VM, nothing critical running)
- Display artifacts during the upgrade (graphics stack replaced while running) → fixed by a reboot

---

## ubuntu-target – Target VM (CIS hardening)

| Field | Value |
|-------|-------|
| Purpose | Target for CIS hardening (before/after comparison, Lynis) |
| OS / Version | Ubuntu Server 26.04.1 LTS |
| Source | ubuntu.com – server ISO |
| SHA256 verified | yes - 2026-10-15 |
| vCPU / RAM / Disk | _2 / 2 GB / TBD_ |
| Network | default (updates) → lab-isolated |
| IP (lab-isolated) | _TBD_ |
| Snapshots | _e.g. `pre-hardening` – date_ |
| Status | _planned / running / done_ |

**Notes / Issues**
- _..._

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
| Status | _planned / running / done_ |

**Notes / Issues**
- _..._
