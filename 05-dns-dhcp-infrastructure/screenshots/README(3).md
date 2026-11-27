# Active Directory DNS & DHCP Infrastructure Lab

![Windows Server 2019](https://img.shields.io/badge/Windows%20Server-2019-0078D6?logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active%20Directory-corp.local-blue)
![DNS](https://img.shields.io/badge/Service-DNS-success)
![DHCP](https://img.shields.io/badge/Service-DHCP-success)
![Kali Linux](https://img.shields.io/badge/Client-Kali%20Linux-557C94?logo=kalilinux&logoColor=white)
![VMware](https://img.shields.io/badge/Hypervisor-VMware%20Workstation-607078?logo=vmware&logoColor=white)

A hands-on Windows Server networking lab covering the deployment, configuration, and validation of **DNS and DHCP services** inside an Active Directory domain (`corp.local`). Everything runs on a Windows Server 2019 Domain Controller (`DC01.corp.local`) and is tested from both a **Windows 10** client and a **Kali Linux** client to confirm cross-platform name resolution and IP lease management.

---

## Table of Contents

- [Objectives](#objectives)
- [Lab Architecture & Environment](#lab-architecture--environment)
- [DNS Configuration & Verification](#dns-configuration--verification)
- [DHCP Configuration & Verification](#dhcp-configuration--verification)
- [Command Reference](#command-reference)
- [Verification Summary](#verification-summary)
- [Notes & Observations](#notes--observations)
- [Skills & Tools Demonstrated](#skills--tools-demonstrated)
- [Repository Structure](#repository-structure)

---

## Objectives

- Create forward lookup records (Host `A` and Alias `CNAME`) so clients reach resources by name instead of IP.
- Build a reverse lookup zone with Pointer (`PTR`) records and verify reverse resolution.
- Confirm that a non-Windows client (Kali Linux) receives valid, authoritative answers from the internal DNS server.
- Deploy and activate a DHCP scope, create a client reservation, and verify lease delivery on a domain-joined Windows 10 client.

---

## Lab Architecture & Environment

| Component | Role / Network Details |
| :--- | :--- |
| **Windows Server 2019 (`DC01.corp.local`)** | Primary Domain Controller, DNS Server, and DHCP Server (`192.168.184.134`) |
| **Windows 10 (`DESKTOP-IBEB2I4`)** | Domain-joined DHCP client and Windows verification workstation (`192.168.184.151`) |
| **Kali Linux (`kali`)** | Independent Linux client for cross-platform DNS query verification |
| **Active Directory Domain** | `corp.local` |
| **Subnet & Gateway** | `192.168.184.0/24` (Default Gateway: `192.168.184.2`) |
| **Hypervisor** | VMware Workstation |

---

## DNS Configuration & Verification

### 1. Forward Lookup Zone: Host (A) Record

Configured a static Host (`A`) record in the `corp.local` forward lookup zone so internal clients can reach the file server by hostname instead of an IP address.

- **Record:** `Fileserver.corp.local` → `192.168.184.134`

![DNS Host A Record Created](screenshots/01_dns_record_created.png)

---

### 2. Alias (CNAME) Record

Added a Canonical Name (`CNAME`) alias pointing to the file server's primary hostname, giving users a simpler, service-oriented share name.

- **Alias:** `FileShare.corp.local` → `Fileserver.corp.local`

![DNS CNAME Alias Created](screenshots/02_Cname_alias_created.png)

---

### 3. Reverse DNS Verification (`nslookup`)

Tested reverse name resolution from the Domain Controller command line to confirm that querying the server's IPv4 address returns a registered hostname.

```cmd
nslookup 192.168.184.134
```

- **Outcome:** `192.168.184.134` resolved back to `Fileserver.corp.local`.

![Reverse DNS Lookup Verification](screenshots/03-reverse-dns-ptr-nslookup.png)

---

### 4. Reverse Lookup Zone & Pointer (PTR) Records

Built the IPv4 reverse lookup zone for the `192.168.184.0/24` subnet and verified Pointer (`PTR`) records for the Domain Controller and the Windows 10 workstation.

- **Reverse Zone:** `184.168.192.in-addr.arpa`
- **PTR Records:**
  - `192.168.184.134` → `DC01.corp.local.`
  - `192.168.184.151` → `DESKTOP-IBEB2I4.corp.local.`

![Reverse Lookup Zone and PTR Records](screenshots/04_reverse-lookup-created.png)

---

### 5. Cross-Platform DNS Verification from Kali Linux (`dig`)

Queried `DC01` directly from a Kali Linux machine with `dig` to confirm that a non-Windows host receives valid, authoritative answers from the internal DNS server.

```bash
dig @192.168.184.134 Fileserver.corp.local
```

- **Outcome:** `status: NOERROR` with the authoritative answer flag (`aa`) set, resolving `Fileserver.corp.local` to `192.168.184.134` (TTL 3600).

![Kali Linux DNS Verification](screenshots/05_kali-dns-verification.png)

---

## DHCP Configuration & Verification

### 6. DHCP IPv4 Scope (`Corp-LAN`)

Created and activated an IPv4 DHCP scope on `DC01` to automate address assignment and distribute gateway and DNS settings across the subnet.

- **Scope Name:** `Corp-LAN` (`192.168.184.0`)
- **Address Pool:** `192.168.184.150` – `192.168.184.200`

![DHCP Scope Address Pool](screenshots/06-dhcp-scope.png)

---

### 7. DHCP Client Reservation

Configured a DHCP reservation inside the `Corp-LAN` scope so a designated workstation can be assigned a fixed IPv4 address while still receiving DNS and gateway options dynamically.

- **Reservation Name:** `Windows10-Client`
- **Reserved IP:** `192.168.184.150`

![DHCP Client Reservation](screenshots/07-dhcp-reservation.png)

---

### 8. Client Lease Renewal & Network Verification (`ipconfig`)

Forced a DHCP lease renewal on the Windows 10 client (`DESKTOP-IBEB2I4`) and inspected its full TCP/IP configuration to confirm that scope options were delivered end to end.

```cmd
ipconfig /renew
ipconfig /all
```

- **Assigned IPv4 Address:** `192.168.184.151` (Subnet Mask: `255.255.255.0`)
- **Default Gateway:** `192.168.184.2`
- **DHCP & DNS Server:** `192.168.184.134`
- **DNS Suffix:** `corp.local`
- **Lease:** obtained Nov 23, 2026 10:54:59 AM, expires Nov 24, 2026 10:54:58 AM

![DHCP Client Renewal Confirmation](screenshots/08-dhcp-renewal-confirmation.png)

---

## Command Reference

Every command used in this lab, with the shell it runs in, what it means, why it was used, and a short summary ready to paste into documentation.

### `nslookup 192.168.184.134`

- **Shell:** Windows Command Prompt (`cmd`)
- **What it means:** `nslookup` queries a DNS server. Given an IP address instead of a name, it performs a **reverse lookup** (PTR query).
- **Purpose:** Verify the reverse zone is working by checking that the server's IP resolves back to a hostname.
- **Documentation summary:** *Ran `nslookup 192.168.184.134` on DC01 to confirm reverse DNS. The query returned `Fileserver.corp.local`, confirming PTR resolution for the 192.168.184.0/24 subnet.*

### `dig @192.168.184.134 Fileserver.corp.local`

- **Shell:** Bash / Zsh (Kali Linux terminal)
- **What it means:** `dig` (Domain Information Groper) sends a DNS query. `@192.168.184.134` forces the query to go to that specific server, and `Fileserver.corp.local` is the name being resolved (an `A` record by default).
- **Purpose:** Prove that a non-Windows client gets a correct, authoritative answer directly from the Domain Controller's DNS service.
- **Documentation summary:** *From Kali Linux, queried DC01 (192.168.184.134) with `dig` for `Fileserver.corp.local`. Response was `NOERROR` with the `aa` flag set and answer `192.168.184.134`, confirming cross-platform DNS resolution.*

### `ipconfig /renew`

- **Shell:** Windows Command Prompt (`cmd`)
- **What it means:** Asks the DHCP server to renew the lease on every adapter that uses DHCP.
- **Purpose:** Force the client to request a fresh lease so the scope, gateway, and DNS options can be observed immediately.
- **Documentation summary:** *Ran `ipconfig /renew` on the Windows 10 client to force a DHCP lease renewal. The client received 192.168.184.151 with the correct mask and gateway.*

### `ipconfig /all`

- **Shell:** Windows Command Prompt (`cmd`)
- **What it means:** Displays the full TCP/IP configuration for all adapters, including MAC address, DHCP server, DNS servers, and lease times.
- **Purpose:** Confirm that DHCP delivered the expected options (IP, mask, gateway, DNS server, DNS suffix, lease duration).
- **Documentation summary:** *Ran `ipconfig /all` to verify DHCP results: IPv4 192.168.184.151, gateway 192.168.184.2, DHCP and DNS server 192.168.184.134, DNS suffix corp.local, one-day lease.*

---

## Verification Summary

| Service / Test | Verified Outcome |
| :--- | :--- |
| **DNS Forward Lookup (A)** | `Fileserver.corp.local` resolves to `192.168.184.134` |
| **DNS Alias (CNAME)** | `FileShare.corp.local` points to `Fileserver.corp.local` |
| **Reverse DNS (`nslookup`)** | `192.168.184.134` resolves back to `Fileserver.corp.local` |
| **Reverse Zone & PTRs** | `184.168.192.in-addr.arpa` active with PTRs for `DC01` and `DESKTOP-IBEB2I4` |
| **Linux DNS Query (`dig`)** | Authoritative `NOERROR` response verified from Kali Linux |
| **DHCP Scope (`Corp-LAN`)** | Active address pool `192.168.184.150` – `192.168.184.200` on `DC01` |
| **DHCP Reservation** | `Windows10-Client` reservation created for `192.168.184.150` |
| **DHCP Client Lease** | `DESKTOP-IBEB2I4` leased `192.168.184.151` with correct DNS and gateway options |

---

## Notes & Observations

- **`.local` naming:** The `dig` output warns that `.local` is reserved for multicast DNS (mDNS). This is acceptable in an isolated lab, but production environments should use a registered domain or a subdomain of one (for example `ad.example.com`).
- **Reservation vs. lease:** The `Windows10-Client` reservation targets `192.168.184.150`, while the Windows 10 client holds `192.168.184.151` from the dynamic pool. A reservation applies only to the client whose MAC address it is bound to.
- **Dynamic DNS records:** The `DESKTOP-IBEB2I4` host record in the forward zone is a dynamically registered, timestamped entry, unlike the static `Fileserver` and `dc01` records.

---

## Skills & Tools Demonstrated

- **Systems Administration:** Windows Server 2019, Active Directory Domain Services (AD DS), DNS Manager, DHCP Manager
- **Networking & Protocols:** IPv4 subnetting, forward and reverse DNS zones (`A`, `CNAME`, `NS`, `SOA`, `PTR`), DHCP scopes, reservations, and lease management
- **Troubleshooting & Diagnostics:** `nslookup`, `dig`, `ipconfig`, cross-platform validation (Windows 10 and Kali Linux)
- **Virtualization:** VMware Workstation lab networking

---

## Repository Structure

```text
.
├── README.md
└── screenshots/
    ├── 01_dns_record_created.png
    ├── 02_Cname_alias_created.png
    ├── 03-reverse-dns-ptr-nslookup.png
    ├── 04_reverse-lookup-created.png
    ├── 05_kali-dns-verification.png
    ├── 06-dhcp-scope.png
    ├── 07-dhcp-reservation.png
    └── 08-dhcp-renewal-confirmation.png
```
