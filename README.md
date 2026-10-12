# NEST 2026 Cohort Final Project

> Secure lab build, assessment and hardening report: an isolated virtual environment with an Active Directory domain controller, a Linux web server, a Windows client and a Kali Linux host.

![Status](https://img.shields.io/badge/Status-Lab%20Environment%20Only-brightgreen)
![Type](https://img.shields.io/badge/Type-Home--Lab-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange)
![Author](https://img.shields.io/badge/Author-0x1aerixis-black)

<p align="center">
  <img src="figures/Report_front_page.png" alt="Front page of the NEST 2026 Cohort Final Project report, titled Secure Lab Build, Assessment and Hardening Report, authored by Michael Asante Anim, dated October 11, 2026" width="420">
</p>
<p align="center"><em>Caption: Front page of the NEST 2026 Cohort Final Project report (October 11, 2026).</em></p>

---

## Table of Contents

- [Overview](#overview)
- [Scope & Objectives](#scope--objectives)
  - [Objectives](#objectives)
  - [In Scope](#in-scope)
  - [Out of Scope](#out-of-scope)
  - [Engagement Type](#engagement-type)
- [Methodology](#methodology)
  - [Network Topology](#network-topology)
  - [VM Inventory](#vm-inventory)
  - [Active Directory Configuration](#active-directory-configuration)
  - [WEB01 Network Configuration](#web01-network-configuration)
- [Findings / Results](#findings--results)
  - [Limitation L-01: Domain Join Blocked](#limitation-l-01-domain-join-blocked)
- [Tools & Environment](#tools--environment)
- [Evidence](#evidence)
  - [Data Handling](#data-handling)
- [Lessons Learned](#lessons-learned)
- [References](#references)
- [Author](#author)

---

## Overview

Organizations rely on directory services and web servers that must be built correctly before they can be secured, tested and defended. A misconfigured network or identity layer undermines every control placed above it.

This project documents the construction of a four-system virtual lab on an isolated network. The lab includes a Windows Server 2022 domain controller, an Ubuntu web server, a Windows 10 client and a Kali Linux workstation. Each system was configured, connected and validated, and the work is recorded as an individual cohort report.

> **Key Outcome:** Built and validated a segmented four-host lab with a working Active Directory domain (`michael.corp`), directory accounts and groups, and verified connectivity between Linux and Windows hosts.

---

## Scope & Objectives

### Objectives

- Build an isolated virtual lab with no route to production networks.
- Deploy an Active Directory domain with organizational units, user accounts and a security group.
- Provision a Linux web server (`WEB01`) with static addressing on the lab segment.
- Configure a Windows 10 client to use the domain controller for DNS.
- Validate connectivity between all lab roles and document each step with evidence.

### In Scope

| Target | Description | Type |
|--------|-------------|------|
| `DC01` (192.168.56.30) | Windows Server 2022 domain controller, domain `michael.corp` | Directory services |
| `WEB01` (192.168.56.40) | Ubuntu Server web server | Linux server |
| `WIN-CLIENT` (192.168.56.20) | Windows 10 Home 22H2 client | Workstation |
| Kali Linux (192.168.56.10) | Assessment workstation | Attacker/analyst host |

### Out of Scope

- Any system outside the isolated virtual lab, including production and university networks.
- Domain join of `WIN-CLIENT`, which was blocked by the installed Windows edition (see [Findings / Results](#findings--results)).
- Internet-facing exposure of any lab service.

### Engagement Type

> **Type:** White-box (all systems are owned and administered by the author)
> **Authorization:** Self-hosted lab; all testing performed only against systems owned by the author
> **Duration:** NEST 2026 cohort project period

---

## Methodology

The work follows the planning, discovery and reporting structure of **NIST SP 800-115** (Technical Guide to Information Security Testing and Assessment), applied to a build-and-validate workflow.

| Phase | Activity | Output |
|-------|----------|--------|
| 1. Planning | Define lab roles, addressing plan and network separation | Topology and IP plan |
| 2. Build | Install and configure `DC01`, `WEB01`, `WIN-CLIENT` and Kali | Running virtual machines |
| 3. Identity configuration | Create OU, users and security group in Active Directory | Directory objects |
| 4. Network configuration | Assign static addresses, set DNS to the domain controller | Reachable lab segment |
| 5. Validation | Test connectivity between hosts and record output | Ping and configuration evidence |
| 6. Reporting | Document steps, results and limitations in a LaTeX report | Final report |

### Network Topology

```text
                  Host-only network: 192.168.56.0/24 (isolated)

   +----------------+      +----------------+      +----------------+      +----------------+
   |  Kali Linux    |      |  WIN-CLIENT    |      |     DC01       |      |     WEB01      |
   |  eth1          |      |  Ethernet1     |      |  Windows       |      |  Ubuntu Server |
   |  .56.10        |      |  .56.20        |      |  Server 2022   |      |  ens37         |
   |                |      |  (Win 10 Home) |      |  .56.30        |      |  .56.40        |
   +-------+--------+      +-------+--------+      +-------+--------+      +-------+--------+
           |                       |                       |                       |
           +-----------------------+-----------+-----------+-----------------------+
                                               |
                                      Virtual host-only switch

   NAT adapters (internet access for updates only):
   Kali eth0 192.168.128.128 | WIN-CLIENT Ethernet0 192.168.128.129 | WEB01 ens33 (DHCP)
```

### VM Inventory

| Hostname | Operating System | Host-only Address | NAT Address | Role |
|----------|------------------|-------------------|-------------|------|
| `DC01` | Windows Server 2022 | 192.168.56.30 | Not applicable | Domain controller, DNS |
| `WIN-CLIENT` | Windows 10 Home 22H2 | 192.168.56.20/24 | 192.168.128.129 | Domain client (join pending) |
| `WEB01` | Ubuntu Server | 192.168.56.40/24 (static) | DHCP on `ens33` | Web server |
| Kali Linux | Kali Linux | 192.168.56.10 | 192.168.128.128 | Assessment workstation |

### Active Directory Configuration

| Item | Value |
|------|-------|
| Root domain | `michael.corp` |
| NetBIOS name | `MICHAEL` |
| Organizational unit | `labUsers` |
| User accounts | `Aerixis`, `hadmin`, `svc_web` |
| Security group | `Staff` (all three accounts are members) |

### WEB01 Network Configuration

Static addressing on the host-only adapter was configured with Netplan in `/etc/netplan/50-cloud-init.yaml`, with `ens33` kept on DHCP for NAT access.

```bash
sudo netplan apply
ip -br address
ping -c 4 192.168.56.20
```

---

## Findings / Results

| ID | Area | Result | Status |
|----|------|--------|--------|
| R-01 | Network | Kali and `WIN-CLIENT` reach each other over `192.168.56.0/24` | Verified |
| R-02 | Linux server | `WEB01` reaches `WIN-CLIENT` (reply TTL 128, consistent with a Windows host) | Verified |
| R-03 | Identity | OU `labUsers`, three accounts and group `Staff` created on `DC01` | Verified |
| R-04 | Client DNS | `WIN-CLIENT` uses `192.168.56.30` as preferred DNS | Configured |
| L-01 | Client domain join | Domain join to `michael.corp` could not be completed | Limitation |

### Limitation L-01: Domain Join Blocked

| Field | Detail |
|-------|--------|
| Affected component | `WIN-CLIENT` (Windows 10 Home 22H2) |
| Description | Windows 10 Home does not support joining an Active Directory domain. The join option is unavailable in this edition. |
| Impact | Domain sign-in and domain-based policy testing on the client were not performed. |
| Resolution | Use Windows 10 Pro, Education or Enterprise for the client, then repeat the join and domain sign-in validation. |
| Retest criteria | `WIN-CLIENT` appears under Computers in Active Directory Users and Computers, and a `MICHAEL\` account signs in successfully. |

Evidence: `figures/win10-home-domain-join-blocked.png`

---

## Tools & Environment

| Category | Tool / Platform | Purpose |
|----------|-----------------|---------|
| Directory services | Windows Server 2022, Active Directory Domain Services, Active Directory Users and Computers | Domain, OU, users, groups |
| Server OS | Ubuntu Server, Netplan | Web server and static addressing |
| Client OS | Windows 10 Home 22H2 | Domain client |
| Assessment host | Kali Linux | Connectivity validation and testing |
| Command-line checks | `ping`, `ip`, `net user`, `net group` | Validation of network and directory state |
| Reporting | LaTeX | Final report authoring |
| Version control | Git, GitHub | Repository and change history |

---

## Evidence

Screenshots are stored in the `figures/` directory.

| File | Shows |
|------|-------|
| `figures/Report_front_page.png` | Front page of the final report |
| `figures/dc_users.png` | Users created in OU `labUsers` (Active Directory Users and Computers) |
| `figures/dc_groups.png` | `net user` and `net group` output confirming accounts and `Staff` group |
| `figures/web01_install_complete.png` | Ubuntu Server installation completed on `WEB01` |
| `figures/configured_the_adapters.png` | Netplan configuration for `ens33` and `ens37` |
| `figures/ping_windows_client.png` | Ping from `WEB01` to `WIN-CLIENT` (192.168.56.20) |
| `figures/win10-home-domain-join-blocked.png` | Domain join unavailable on Windows 10 Home |

> Note: Screenshots taken on `DC01` may show the default Windows hostname. The report refers to the server as `DC01`.

### Data Handling

- All hostnames, accounts and data exist only inside the isolated virtual lab.
- No real passwords or personal information are published in this repository.
- Lab IP addresses are private RFC 1918 ranges with no route to production networks.

---

## Lessons Learned

| Topic | Takeaway |
|-------|----------|
| Windows editions | Domain join requires Pro, Education or Enterprise. Edition checks belong in the planning phase, before the client is built. |
| DNS dependency | A domain client must use the domain controller as its DNS server, or domain discovery fails. |
| Netplan | Static addressing on a second adapter requires correct YAML indentation and an explicit `netplan apply`, and must not disturb the DHCP-configured NAT adapter. |
| Network separation | A host-only segment combined with a NAT adapter limits exposure while still allowing package updates. |
| Documentation | Capturing a screenshot and command output at each step keeps every claim in the report verifiable. |

**Skills demonstrated:** `Active Directory` · `Windows Server 2022` · `Linux Administration` · `Netplan` · `Network Segmentation` · `Virtual Lab Design` · `Technical Reporting` · `LaTeX`

---

## References

- NIST SP 800-115, Technical Guide to Information Security Testing and Assessment: https://csrc.nist.gov/pubs/sp/800/115/final
- Microsoft, Active Directory Domain Services overview: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview
- Netplan documentation: https://netplan.io/
- RFC 1918, Address Allocation for Private Internets: https://datatracker.ietf.org/doc/html/rfc1918

---

## Author

**Michael Asante Anim** | `0x1aerixis`
BSc Cyber Security — University of Mines and Technology (UMaT), Tarkwa, Ghana

[![GitHub](https://img.shields.io/badge/GitHub-anim--michael--asante-black?logo=github)](https://github.com/anim-michael-asante)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://linkedin.com/in/michael-asante-anim)
[![TryHackMe](https://img.shields.io/badge/TryHackMe-0x1aerixis-red?logo=tryhackme)](https://tryhackme.com/p/0x1aerixis)
[![X](https://img.shields.io/badge/X-0x1aerixis-black?logo=x)](https://x.com/0x1aerixis)
[![Discord](https://img.shields.io/badge/Discord-0x1aerixis-5865F2?logo=discord)](https://discord.com/users/0x1aerixis)

> *"Built in the lab. Documented for the field."*

---
> **Disclaimer:** All work documented in this repository was conducted in authorized, isolated
> lab environments or sanctioned CTF platforms. No unauthorized systems were accessed.
> This project is intended for educational and portfolio purposes only.