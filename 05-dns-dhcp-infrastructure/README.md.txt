# Active Directory DNS & DHCP Configuration Lab

A hands-on Windows Server networking lab focused on configuring and verifying **DNS and DHCP services** in an Active Directory environment.

The lab was built around a Windows Server Domain Controller (`DC01.corp.local`) and verified from both a **Windows 10 client** and a **Kali Linux client**.

---

## Lab Environment

| Component | Role / Details |
|---|---|
| **Windows Server (`DC01.corp.local`)** | Primary Domain Controller, DNS Server, and DHCP Server (`192.168.184.134`) |
| **Windows 10 (`DESKTOP-IBEB2I4`)** | DHCP client and Windows verification client (`192.168.184.151`) |
| **Kali Linux** | Independent Linux DNS verification client |
| **Active Directory Domain** | `corp.local` |
| **Virtualization Platform** | VMware Workstation |

---

## DNS Configuration & Verification

### 1. DNS Forward Lookup — A Record

**Purpose:** Configure DNS name resolution for a file server so clients can locate it by hostname rather than relying solely on an IP address.

* **Configuration:** `Fileserver.corp.local` → `192.168.184.134`

![DNS A Record Created](./01_dns_record_created.png)

> **Result:** The static `Fileserver` Host (`A`) record was successfully created in the `corp.local` forward lookup zone.

---

### 2. DNS CNAME Alias

**Purpose:** Create an easy-to-remember DNS alias that points to the existing `Fileserver` hostname.

* **Configuration:** `FileShare.corp.local` → `Fileserver.corp.local`

![DNS CNAME Alias Created](./02_Cname_alias_created.png)

> **Result:** The `FileShare` Alias (`CNAME`) record was successfully configured to resolve to `Fileserver.corp.local`.

---

### 3. Reverse DNS Lookup Verification

**Purpose:** Verify that DNS can perform **reverse name resolution** by resolving an IP address back to its hostname.

```cmd
nslookup 192.168.184.134
```

![Reverse DNS PTR nslookup](./03-reverse-dns-ptr-nslookup.png)

> **Result:** The IP address `192.168.184.134` successfully resolves back to `Fileserver.corp.local`.

---

### 4. Reverse Lookup Zone & PTR Records

**Purpose:** Configure a reverse lookup zone so IPv4 addresses on the subnet map to their corresponding hostnames.

* **Reverse Zone:** `184.168.192.in-addr.arpa`
* **PTR Records Verified:**
  * `192.168.184.134` → `DC01.corp.local`
  * `192.168.184.151` → `DESKTOP-IBEB2I4.corp.local`

![Reverse Lookup Zone Created](./04_reverse-lookup-created.png)

> **Result:** The reverse lookup zone and Pointer (`PTR`) records are active and resolving hosts on the lab subnet.

---

### 5. DNS Verification from Kali Linux

**Purpose:** Verify cross-platform DNS resolution from an independent Linux client rather than relying only on local Windows Server tests.

```bash
dig @192.168.184.134 Fileserver.corp.local
```

![Kali DNS Verification](./05_kali-dns-verification.png)

> **Result:** The query returned an authoritative (`aa`) response with a `NOERROR` status, confirming `Fileserver.corp.local` resolves to `192.168.184.134`.

---

## DHCP Configuration & Verification

### 6. DHCP IPv4 Scope

**Purpose:** Configure a DHCP scope to dynamically assign IPv4 addresses and network settings to clients on the lab network.

* **Scope Name:** `Corp-LAN`
* **Network ID:** `192.168.184.0/24`
* **Address Pool:** `192.168.184.150 – 192.168.184.200`

![DHCP Scope](./06-dhcp-scope.png)

> **Result:** The `Corp-LAN` IPv4 DHCP scope was configured and activated on `dc01.corp.local`.

---

### 7. DHCP Client Reservation

**Purpose:** Reserve a dedicated IP address for a specific workstation while still managing its network configuration centrally through DHCP.

* **Client Name:** `Windows10-Client`
* **Reserved IP:** `192.168.184.150`

![DHCP Reservation](./07-dhcp-reservation.png)

> **Result:** A DHCP reservation for `192.168.184.150` was created under the `Corp-LAN` scope.

---

### 8. DHCP Client Renewal & Configuration Verification

**Purpose:** Verify that the Windows 10 client (`DESKTOP-IBEB2I4`) communicates with the DHCP server and leases the expected network configuration.

```cmd
ipconfig /renew
ipconfig /all
```

* **Verified Lease Details:**
  * **IPv4 Address:** `192.168.184.151`
  * **Default Gateway:** `192.168.184.2`
  * **DHCP & DNS Server:** `192.168.184.134`

![DHCP Renewal Confirmation](./08-dhcp-renewal-confirmation.png)

> **Result:** The Windows 10 client successfully renewed its DHCP lease and received the proper gateway and DNS server options.

---

## Verification Summary

| Service | Verification Outcome |
|---|---|
| **DNS A Record** | `Fileserver.corp.local` resolves to `192.168.184.134` |
| **DNS CNAME** | `FileShare.corp.local` resolves to `Fileserver.corp.local` |
| **Reverse DNS** | `nslookup` resolves `192.168.184.134` back to hostname |
| **PTR Records** | `184.168.192.in-addr.arpa` zone populated with host PTR records |
| **Kali Linux DNS** | `dig` confirmed authoritative `NOERROR` resolution from Linux |
| **DHCP Scope** | `Corp-LAN` (`192.168.184.150 – 200`) active on `DC01` |
| **DHCP Reservation** | Static lease (`192.168.184.150`) configured in DHCP Manager |
| **DHCP Client** | Windows 10 renewed lease (`192.168.184.151`) with DNS & Gateway |

---

## Technologies & Tools Used

* **Operating Systems:** Windows Server 2019, Windows 10, Kali Linux
* **Infrastructure Services:** Active Directory Domain Services (AD DS), DNS, DHCP
* **Virtualization:** VMware Workstation
* **CLI Verification Tools:** `nslookup`, `dig`, `ipconfig`

---

## Project Objective & Outcome

The objective of this lab was to gain practical experience deploying, managing, and troubleshooting core Windows Server network infrastructure services in a virtualized environment.

By verifying configurations from both **Windows 10** and **Kali Linux**, the lab demonstrates a functional, cross-platform enterprise network foundation with centralized **DNS name resolution** and **automated DHCP IP management**.
