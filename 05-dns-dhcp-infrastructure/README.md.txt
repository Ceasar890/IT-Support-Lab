# DNS & DHCP Infrastructure Configuration

**Environment:** VMware Workstation Pro | **Domain:** `corp.local` | **Subnet:** `192.168.184.0/24`

## Project Overview
This lab covers the setup, verification, and troubleshooting of core network services—DNS and DHCP—on a Windows Server Domain Controller (`DC01`). To ensure cross-platform reliability across a mixed environment, all records and IP leases were validated from both a domain-joined Windows 10 client and a Kali Linux host.

## Lab Topology

| Hostname / VM | OS / Role | IP Address | Notes |
| :--- | :--- | :--- | :--- |
| **DC01** | Windows Server (AD DS, DNS, DHCP) | `192.168.184.134` | Primary domain controller and network services host |
| **DESKTOP-IBEB2I4** | Windows 10 Client | `192.168.184.151` (DHCP) | Domain-joined workstation (`00-0C-29-5A-64-D7`) |
| **kali** | Kali Linux | DHCP / Static (`192.168.184.x`) | External Linux client used for `dig` DNS queries |

## Key Objectives
* Configure forward lookup records (`A` and `CNAME`) in Windows DNS Manager.
* Build an Active Directory-integrated Reverse Lookup Zone and verify `PTR` resolution.
* Validate DNS resolution across both Windows (`nslookup`) and Linux (`dig`) clients.
* Configure a DHCP IPv4 scope, create a MAC-based reservation, and verify client lease renewal.

---

## Implementation Walkthrough

### 1. Forward DNS: Host (A) Record Creation
Created a static Host (`A`) record for an internal file server inside the `corp.local` forward lookup zone so internal clients can resolve the server by hostname rather than IP.
* **Record:** `Fileserver.corp.local` → `192.168.184.134`
* **Screenshot:** `screenshots/01_dns_record_created.png`

### 2. Forward DNS: Alias (CNAME) Record
Added a `CNAME` alias pointing `FileShare.corp.local` to the `Fileserver.corp.local` host record. This allows users and mapped drives to reference a service-friendly name even if the underlying host changes later.
* **Alias:** `FileShare.corp.local` → `Fileserver.corp.local`
* **Screenshot:** `screenshots/02_Cname_alias_created.png`

### 3. Reverse DNS Lookup Verification (`nslookup`)
Tested reverse name resolution from Command Prompt on `DC01` by querying the server IP directly (`nslookup 192.168.184.134`), confirming it resolves cleanly back to `Fileserver.corp.local`.
* **Screenshot:** `screenshots/03-reverse-dns-ptr-nslookup.png`

### 4. Reverse Lookup Zone & PTR Configuration
Configured the IPv4 Reverse Lookup Zone (`184.168.192.in-addr.arpa`) and verified the associated Pointer (`PTR`) record for `192.168.184.134` in DNS Manager.
* **Zone:** `184.168.192.in-addr.arpa`
* **Screenshot:** `screenshots/04_reverse_lookup_created.png`

### 5. Cross-Platform DNS Validation (Kali Linux)
Queried `DC01` directly from a Kali Linux VM using `dig @192.168.184.134 Fileserver.corp.local`. The server returned an authoritative `NOERROR` response (`flags: qr aa rd ra`) with a `0 msec` query time, confirming Linux clients resolve internal AD DNS records properly.
* **Screenshot:** `screenshots/05_kali-dns-verification.png`

### 6. DHCP Scope Configuration
Configured an active IPv4 DHCP scope (`Scope Corp-LAN`) on `DC01` to dynamically assign addresses to network clients.
* **Address Pool Range:** `192.168.184.150` – `192.168.184.200`
* **Subnet Mask:** `255.255.255.0`
* **Screenshot:** `screenshots/06-dhcp-scope.png`

### 7. DHCP Reservation Setup
Created a DHCP reservation (`Windows10-Client`) tied to the workstation's MAC address so critical endpoints retain a predictable IP address across reboots and lease renewals.
* **Screenshot:** `screenshots/07-dhcp-reservation.png`

### 8. Client Lease Renewal & Verification
Ran `ipconfig /renew` and `ipconfig /all` on the Windows 10 client (`DESKTOP-IBEB2I4`). Confirmed the adapter successfully leased an IP from `DC01` (`192.168.184.134`) with the `corp.local` DNS suffix, default gateway (`192.168.184.2`), and DNS server settings applied automatically.
* **Screenshot:** `screenshots/08-dhcp-renewal-confirmation.png`

---

## Summary of Skills Demonstrated
* **Windows Server Administration:** Managing Active Directory-integrated DNS zones (`A`, `CNAME`, `PTR`) and IPv4 DHCP scopes/reservations.
* **Network Troubleshooting & Validation:** Using `nslookup`, `dig`, and `ipconfig` to verify name resolution, authoritative DNS flags, and DHCP lease delivery.
* **Multi-OS Interoperability:** Validating Windows infrastructure services from a Linux endpoint within a virtualized lab network.
