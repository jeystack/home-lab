# Lab Notes

Personal notes from my home lab and Linux setup. I document what I built, what broke, how I fixed it and what I learned.

---

## 2026-10-14 – Dual boot: Ubuntu 26.04.1 LTS + Windows 11

### Setup
- **Hardware:** Lenovo ThinkPad T14 Gen 2 (Intel i7-1185G7, 32 GB RAM, 1 TB NVMe SSD)
- **Partitioning:** Shrank the Windows partition (C:) to ~229 GB from within Windows Disk Management, leaving ~700 GB unallocated for Linux
- **Windows encryption:** BitLocker suspended before the installation, re-enabled afterwards
- **Linux encryption:** Full disk encryption with LUKS + LVM, ext4 root file system, separate unencrypted `/boot` partition (needed so the system can boot and ask for the passphrase)
- **Bootloader:** GRUB with Ubuntu and Windows Boot Manager, menu timeout set to 10 seconds
- **Secure Boot:** Left disabled for now (BIOS showed key state "Customized" – to be checked later)
- **Firewall:** `ufw` enabled – deny incoming, allow outgoing
- **Snapshots:** Timeshift (RSYNC), 5 daily + 3 weekly snapshots

### Decisions
- **Ubuntu as host, Kali only as a VM** – a stable daily system on the host, attack tools isolated in a virtual machine that can be reset at any time
- **Disk encryption on both systems** – the laptop holds SSH keys and personal documents
- **Hardware clock (RTC) in UTC** – Linux default; Windows will be switched to UTC via registry. Keeping the RTC in local time causes problems with daylight saving time changes when two operating systems share one clock
- **Firmware updates only via Lenovo Vantage on Windows** – one place for BIOS updates instead of two

### Troubleshooting

#### 1. No internet over Ethernet
- **Symptom:** With Wi-Fi off, `ping 8.8.8.8` returned "Network is unreachable", although the link was up (`LOWER_UP` in `ip link`)
- **Analysis:**
  - `ip route` showed an address in `169.254.0.0/16` (APIPA / link-local) and **no default route** → DHCP was not working
  - `nmcli connection show <profile> | grep ipv4.method` returned `link-local` – the installer had created a connection profile that never requests an address via DHCP
- **Fix:**
  ```bash
  sudo nmcli connection modify <profile> ipv4.method auto
  sudo nmcli connection down <profile>
  sudo nmcli connection up <profile>
  ```
- **Result:** Default route via the router (`192.168.178.1`) assigned by DHCP, `ping google.com` with 0 % packet loss
- **Lesson:** Troubleshoot bottom-up: link → IP address → route → DNS. A `169.254.x.x` address means the DHCP request got no answer

#### 2. USB-C monitor: "No Signal"
- **Symptom:** After several reboots the external monitor showed "No Signal". The laptop screen flickered when plugging in the cable, so the display was detected but the connection was not established
- **Fix:** Disconnected the monitor from mains power for one minute
- **Lesson:** Rule out the simplest cause (hung device) before suspecting drivers or hardware

#### 3. System clock off by two hours
- **Symptom:** After `timedatectl set-local-rtc 1 --adjust-system-clock` the system showed UTC as local time
- **Fix:** Re-synced via NTP (`timedatectl set-ntp false` / `true`), then switched back to UTC (`timedatectl set-local-rtc 0`)
- **Lesson:** Read the warnings – `timedatectl` explicitly recommends keeping the RTC in UTC

### Terminal learnings
- Linux file names are case-sensitive (`Discord.deb` ≠ `discord.deb`)
- If a wildcard (`*`) matches nothing, the shell passes it on literally → check with `ls` first
- **Tab** completes names, **↑** recalls the last command
- `rm` deletes without a trash bin
- `grep` matches exactly – a typo gives no output, not an error
- Commands like `apt update` or `systemctl` work from any directory; `./file` refers to the current directory

#### 5. Ethernet connected, but no gateway and no routes
- **Symptom:** After reconnecting the laptop, traffic only went over Wi-Fi. `nmcli` showed Ethernet as "connected" and the interface had an IPv4 address, but `ip route` listed no routes for `enp0s31f6`
- **Analysis:**
  - `ip addr` showed the address with the flag `noprefixroute` → address present, but no route added
  - `nmcli -f IP4 device show enp0s31f6`: address and DNS received via DHCP, but `IP4.GATEWAY: --` and no `IP4.ROUTE` entries
  - Ruled out: profile settings `never-default`, `ignore-auto-routes` and `route-table` were normal
  - Ruled out: `systemd-networkd` inactive, so no second service removing routes
  - Reconnecting the profile (`down` / `up`) did not help
- **Fix:** Backed up the installer-generated profile, deleted it and created a clean default profile:
```bash
  nmcli connection show <old-profile> > ~/lab/lan-profil-alt.txt
  sudo nmcli connection delete <old-profile>
  sudo nmcli connection add type ethernet ifname enp0s31f6 con-name LAN ipv4.method auto ipv6.method auto
  sudo nmcli connection up LAN
```
- **Result:** Gateway `192.168.178.1`, default route via Ethernet (metric 100) preferred over Wi-Fi (metric 600)
- **Lesson:** Form a hypothesis, test it, discard it, test the next one. Back up a configuration before deleting it. Lower route metric = preferred path

### Open tasks
- [ ] Windows: set `RealTimeIsUniversal` so Windows reads the RTC as UTC
- [ ] Check whether Secure Boot can be enabled safely
- [ ] Rebuild the security lab with KVM/virt-manager (Kali + Ubuntu VM, isolated network)
- [ ] Harden the Ubuntu VM following the CIS Benchmark, document before/after (Lynis)
