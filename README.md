<div align="center">

# 🛡️ Cybersecurity Lab Environment Setup
### VMware Workstation + Kali Linux

![Status](https://img.shields.io/badge/status-completed-brightgreen)
![Platform](https://img.shields.io/badge/hypervisor-VMware%20Workstation-blue)
![Guest OS](https://img.shields.io/badge/guest%20OS-Kali%20Linux%202026.2-557C94)
![Network](https://img.shields.io/badge/network-NAT%2010.0.0.0%2F24-orange)
![Program](https://img.shields.io/badge/Networkwalks-Academy-critical)

**WK1-PM1 — Lab Setup** · Cybersecurity Internship, Week 1
**Author:** Haris Abdullah · **Date:** 1 November 2026

</div>

---

## 📑 Table of Contents

- [1. Project Overview and Objectives](#1-project-overview-and-objectives)
- [2. Purpose of the Lab](#2-purpose-of-the-lab)
- [3. Lab Architecture and Configuration](#3-lab-architecture-and-configuration)
- [4. Setup Procedure](#4-setup-procedure)
- [5. Verification](#5-verification)
- [6. Problems and Solutions](#6-problems-and-solutions)
- [7. Learning Reflections](#7-learning-reflections)
- [8. Security and Ethical Use](#8-security-and-ethical-use)
- [9. Tools and Resources](#9-tools-and-resources)
- [10. Author](#10-author)

---

## 1. Project Overview and Objectives

This project documents the setup of an isolated cybersecurity testing lab on my personal laptop, using **VMware Workstation** (in place of VirtualBox) as the hypervisor and **Kali Linux** as the attacking/testing machine, on a private NAT-based virtual network.

> 🎯 **Objectives**
> - Build a self-contained virtual lab, isolated from the host's main network
> - Configure Kali Linux with a static IP on a custom `10.0.0.0/24` subnet
> - Enable file sharing and clipboard integration between host and guest
> - Verify full internet connectivity from the guest VM
> - Document the setup process, including real issues encountered and how they were resolved

## 2. Purpose of the Lab

This environment will be used for future practice in network scanning, packet analysis, and web security testing, strictly on systems I own or have explicit permission to test.

## 3. Lab Architecture and Configuration

| 🧩 Component | Detail |
|---|---|
| 💻 Host OS | Windows 10/11 |
| 🖥️ Hypervisor | VMware Workstation |
| 🐉 Guest OS | Kali Linux (2026.2) |
| 🌐 Network type | NAT (VMnet8) |
| 📡 Subnet | `10.0.0.0/24` |
| 🏷️ Kali static IP | `10.0.0.2/24` |
| 🚪 Gateway | `10.0.0.1` |
| 🔎 DNS | `8.8.8.8` |
| 📂 Shared folder | Host `Downloads` → mounted at `/mnt/hgfs/Downloads` in Kali |
| 📋 Clipboard / Drag-drop | Enabled (bidirectional) |

## 4. Setup Procedure

1. **Installed VMware Workstation** on the host machine.
2. **Configured the NAT network (VMnet8)** via VMware's Virtual Network Editor:
   - Subnet IP: `10.0.0.0`
   - Subnet mask: `255.255.255.0`
   - Gateway IP (NAT Settings): `10.0.0.1`
3. **Attached the Kali VM's network adapter** to NAT (VMnet8).
4. **Configured Kali's static IP** via NetworkManager:
   ```bash
   sudo nmcli connection modify "Wired connection 1" ipv4.addresses 10.0.0.2/24
   sudo nmcli connection modify "Wired connection 1" ipv4.gateway 10.0.0.1
   sudo nmcli connection modify "Wired connection 1" ipv4.dns "8.8.8.8"
   sudo nmcli connection modify "Wired connection 1" ipv4.method manual
   sudo nmcli connection down "Wired connection 1"
   sudo nmcli connection up "Wired connection 1"
   ```
5. **Enabled Shared Folders and Guest Isolation** (clipboard + drag/drop) in VMware VM Settings → Options.
6. **Installed/confirmed `open-vm-tools-desktop`** and manually mounted the shared folder:
   ```bash
   sudo mkdir -p /mnt/hgfs
   sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
   ```
7. **Took a clean snapshot** ("Working Setup") once the configuration was verified working, as a recovery baseline.

<br>

<div align="center">
<img src="images/2-screenshot-nat-network-settings.png" width="650"/>
<br><sub><b>🖥️ VMware Virtual Network Editor</b> — VMnet8 set to NAT, subnet 10.0.0.0/24</sub>
</div>

<br>

<div align="center">
<img src="images/3-screenshot-kali-network-adapter.png" width="650"/>
<br><sub><b>🔌 Kali VM network adapter</b> — attached to NAT</sub>
</div>

<br>

<div align="center">
<img src="images/4-screenshot-kali-static-ip.png" width="650"/>
<br><sub><b>🏷️ Static IP confirmed</b> — <code>ip addr show</code> output: 10.0.0.2/24 on eth0</sub>
</div>

<br>

<div align="center">
<img src="images/5-screenshot-guest-isolation.png" width="650"/>
<br><sub><b>📋 Guest isolation</b> — clipboard sharing and drag-and-drop enabled</sub>
</div>

<br>

<div align="center">
<img src="images/6-screenshot-shared-folder.png" width="650"/>
<br><sub><b>📂 Shared folder verified</b> — Host Downloads folder mounted at /mnt/hgfs/</sub>
</div>

<br>

## 5. Verification

| ✅ Test | Command | Result |
|---|---|---|
| IP configuration | `ip addr show eth0` | `10.0.0.2/24` confirmed |
| Gateway reachability | `ping -c 4 10.0.0.1` | 🟢 0% packet loss |
| Internet reachability | `ping -c 4 8.8.8.8` | 🟢 0% packet loss |
| DNS resolution | `ping -c 4 tryhackme.com` | 🟢 Resolved and replied |
| Shared folder | `ls /mnt/hgfs/` | 🟢 `Downloads` listed |
| Snapshot | Snapshot → Take Snapshot | 🟢 "Working Setup" created |

## 6. Problems and Solutions

> Documenting problems is an important part of this project, as real-world lab setups rarely go smoothly on the first attempt.

<details>
<summary><b>🔴 Problem 1 — Gateway unreachable after setting Kali's static IP to 10.0.0.2</b></summary>
<br>

After manually assigning Kali the IP `10.0.0.2/24` with gateway `10.0.0.1`, both the gateway and internet became unreachable, and DNS failed with "Temporary failure in name resolution."

**🔍 Diagnosis:** Inspecting VMware's NAT Settings showed the NAT Gateway IP for VMnet8 was itself set to `10.0.0.2` by default — the exact same address assigned to Kali. This caused an IP conflict (VirtualBox's default NAT gateway is `.1`, but VMware's default is `.2`, which the original lab guide didn't account for since it was written for VirtualBox).

**🟢 Fix:** Changed the NAT Gateway IP in Virtual Network Editor → NAT Settings from `10.0.0.2` to `10.0.0.1`, restarted the VMware NAT and DHCP services, and rebooted the host.

</details>

<details>
<summary><b>🔴 Problem 2 — Gateway reachable, but internet (8.8.8.8) and DNS still failing</b></summary>
<br>

After fixing the gateway conflict, `ping 10.0.0.1` succeeded, but `ping 8.8.8.8` and DNS lookups still failed with 100% packet loss.

**🔍 Diagnosis:** Tested in both directions — pinging the Kali guest from the Windows host succeeded (0% loss), but pinging the host from Kali failed, which pointed first at Windows Firewall. Temporarily disabling Windows Firewall entirely did not resolve it, ruling out the firewall and pointing instead to the VMware NAT driver/service being in a broken state after the manual gateway change.

**🟢 Fix:** Used Virtual Network Editor's **Restore Defaults**, then reconfigured VMnet8 back to subnet `10.0.0.0/24` with gateway `10.0.0.1`, and performed a full host reboot (not just a service restart). This fully resolved connectivity — gateway, internet, and DNS all confirmed working afterward.

</details>

<details>
<summary><b>🔴 Problem 3 — Shared folder not auto-mounting at /mnt/hgfs/</b></summary>
<br>

Despite enabling Shared Folders in VM Settings and confirming `open-vm-tools-desktop` was installed, `/mnt/hgfs/` did not exist after reboot.

**🟢 Fix:** Manually mounted it with:
```bash
sudo mkdir -p /mnt/hgfs
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
```
*(Note: the `-o nonempty` option is not supported by this FUSE version and must be omitted.)*

</details>

## 7. Learning Reflections

- 🧠 VMware and VirtualBox differ in their default NAT gateway addressing (`.2` vs `.1`), which can silently cause IP conflicts when following cross-tool guides.
- 🧠 Diagnosing connectivity issues benefits from testing each layer independently: interface IP → gateway → external IP → DNS, and testing traffic in both directions (host↔guest) to isolate firewall vs. driver-level problems.
- 🧠 A full host reboot is sometimes necessary after changing virtual network settings — restarting individual services is not always sufficient.
- 🧠 Taking a snapshot only after confirming a fully working configuration avoids baking broken network state into your recovery point.

## 8. Security and Ethical Use

> ⚠️ This lab environment is isolated on a private NAT network and must only be used for systems I own or have explicit permission to test. It is for educational and ethical cybersecurity practice only.

## 9. Tools and Resources

![VMware](https://img.shields.io/badge/-VMware%20Workstation-607078?style=flat-square)
![Kali](https://img.shields.io/badge/-Kali%20Linux%202026.2-557C94?style=flat-square)
![nmcli](https://img.shields.io/badge/-nmcli%20%2F%20NetworkManager-4E9A06?style=flat-square)
![OVT](https://img.shields.io/badge/-open--vm--tools--desktop-CC0000?style=flat-square)

## 10. Author

<div align="center">

**Haris Abdullah**
Networkwalks Academy — Cybersecurity Internship, Week 1

</div>
