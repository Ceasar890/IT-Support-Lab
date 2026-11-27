# Active Directory DNS & DHCP Infrastructure Lab

A practical Windows Server networking lab focused on deploying, configuring, and validating **DNS and DHCP services** inside an Active Directory domain (`corp.local`). All services were built on a Windows Server 2019 Domain Controller (`DC01.corp.local`) and tested across both **Windows 10** and **Kali Linux** clients to confirm cross-platform name resolution and IP lease management.

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

### 1. Forward Lookup Zone — Host (A) Record
Configured a static Host (A) record inside the `corp.local` forward lookup zone so internal clients can reach the file server by hostname instead of an IP address.

* **Record:** `Fileserver.corp.local` → `192.168.184.134`

![DNS Host A Record Created](01_dns_record_created.png)

---

### 2. Alias (CNAME) Record
Added a Canonical Name (CNAME) alias pointing to the file server's primary hostname to give users a simpler, service-oriented share name.

* **Alias:** `FileShare.corp.local` → `Fileserver.corp.local`

![DNS CNAME Alias Created](02_Cname_alias_created.png)

---

### 3. Reverse DNS Lookup Verification (`nslookup`)
Tested reverse name resolution from the Domain Controller command line to verify that querying the server's IPv4 address returns its registered hostname.

```cmd
nslookup 192.168.184.134
```

* **Outcome:** `192.168.184.134` resolved cleanly back to `Fileserver.corp.local`.

![Reverse DNS Lookup Verification](03-reverse-dns-ptr-nslookup.png)

---

### 4. Reverse Lookup Zone & Pointer (PTR) Records
Built the IPv4 reverse lookup zone for the `192.168.184.0/24` subnet and verified active Pointer (PTR) records for the Domain Controller and Windows 10 workstation.

* **Reverse Zone:** `184.168.192.in-addr.arpa`
* **PTR Records:**
  * `192.168.184.134` → `DC01.corp.local.`
  * `192.168.184.151` → `DESKTOP-IBEB2I4.corp.local.`

![Reverse Lookup Zone and PTR Records](04_reverse-lookup-created.png)

---

### 5. Cross-Platform DNS Verification from Kali Linux (`dig`)
Queried `DC01` directly from an external Kali Linux machine using `dig` to confirm that non-Windows hosts receive valid, authoritative answers from the internal DNS server.

```bash
dig @192.168.184.134 Fileserver.corp.local
```

* **Outcome:** Returned `status: NOERROR` with the authoritative answer flag (`aa`) set, resolving `Fileserver.corp.local` to `192.168.184.134`.

![Kali Linux DNS Verification](05_kali-dns-verification.png)

---

## DHCP Configuration & Verification

### 6. DHCP IPv4 Scope (`Corp-LAN`)
Created and activated an IPv4 DHCP scope on `DC01` to automate address assignment and distribute gateway and DNS settings across the subnet.

* **Scope Name:** `Corp-LAN` (`192.168.184.0`)
* **Address Pool:** `192.168.184.150 – 192.168.184.200`

![DHCP Scope Address Pool](06-dhcp-scope.png)

---

### 7. DHCP Client Reservation
Configured a static DHCP reservation inside the `Corp-LAN` scope so a designated workstation consistently receives the same IPv4 address while still pulling DNS and gateway options dynamically.

* **Reservation Name:** `Windows10-Client`
* **Reserved IP:** `192.168.184.150`

![DHCP Client Reservation](07-dhcp-reservation.png)

---

### 8. Client Lease Renewal & Network Verification (`ipconfig`)
Forced a DHCP lease renewal on the Windows 10 client (`DESKTOP-IBEB2I4`) and inspected its full TCP/IP adapter configuration to confirm end-to-end delivery of scope options.

```cmd
ipconfig /renew
ipconfig /all
```

* **Assigned IPv4 Address:** `192.168.184.151` (Subnet Mask: `255.255.255.0`)
* **Default Gateway:** `192.168.184.2`
* **DHCP & DNS Server:** `192.168.184.134`
* **DNS Suffix:** `corp.local`

![DHCP Client Renewal Confirmation](08-dhcp-renewal-confirmation.png)

---

## Verification Summary

| Service / Test | Verified Outcome |
| :--- | :--- |
| **DNS Forward Lookup (A)** | `Fileserver.corp.local` resolves to `192.168.184.134` |
| **DNS Alias (CNAME)** | `FileShare.corp.local` resolves to `Fileserver.corp.local` |
| **Reverse DNS (`nslookup`)** | `192.168.184.134` resolves back to `Fileserver.corp.local` |
| **Reverse Zone & PTRs** | `184.168.192.in-addr.arpa` active with PTRs for `DC01` and `DESKTOP-IBEB2I4` |
| **Linux DNS Query (`dig`)** | Authoritative `NOERROR` response verified from Kali Linux |
| **DHCP Scope (`Corp-LAN`)** | Active address pool (`192.168.184.150 – 192.168.184.200`) on `DC01` |
| **DHCP Reservation** | Static reservation configured for `Windows10-Client` (`192.168.184.150`) |
| **DHCP Client Lease** | `DESKTOP-IBEB2I4` leased `192.168.184.151` with proper DNS and gateway options |

---

## Skills & Tools Demonstrated

* **Systems Administration:** Windows Server 2019, Active Directory Domain Services (AD DS), DNS Manager, DHCP Manager
* **Networking & Protocols:** IPv4 Subnetting, Forward/Reverse DNS Zones (A, CNAME, NS, SOA, PTR), DHCP Scopes & Reservations
* **Troubleshooting & Diagnostics:** `nslookup`, `dig`, `ipconfig`, cross-platform network validation (Windows 10 & Kali Linux)
* **Virtualization:** VMware Workstation host-only/NAT lab networking
