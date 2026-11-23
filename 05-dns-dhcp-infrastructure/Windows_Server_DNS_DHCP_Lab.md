# Windows Server DNS & DHCP Lab

Welcome to my home lab documentation. In this project, I built and tested core network services using **Windows Server 2019** as the domain controller. The main goal was to understand how **DNS** and **DHCP** work in a small business-style network and to verify that the services worked across different operating systems.

I tested the environment using **Windows 10** and **Kali Linux** virtual machines running in VMware Workstation.

---

## 🖥️ Lab Setup

The lab uses an isolated VMware network:

- **Network:** `192.168.184.0/24`
- **Domain:** `corp.local`
- **Domain Controller:** `DC01`
- **Server OS:** Windows Server 2019
- **Windows Client:** `DESKTOP-IBEB2I4`
- **Linux Client:** Kali Linux

### Main Server

**DC01**

- IP address: `192.168.184.134`
- Services: Active Directory, DNS, DHCP

### Windows 10 Client

The Windows 10 machine was configured as a DHCP client so that it could receive its network configuration automatically.

During final verification, the client received:

- IPv4 address: `192.168.184.151`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.184.2`
- DHCP server: `192.168.184.134`
- DNS server: `192.168.184.134`

---

# 🚀 What I Built and Tested

## 1. Forward DNS Lookup

The first part of the project was configuring DNS so computers could be found using hostnames instead of having to remember IP addresses.

I created the following DNS records:

- **A record:** `dc01.corp.local` → `192.168.184.134`
- **A record:** `Fileserver.corp.local` → `192.168.184.134`
- **CNAME:** `FileShare.corp.local` → `Fileserver.corp.local`

The CNAME gives users a simple, friendly name (`FileShare`) while still pointing to the actual `Fileserver` hostname.

### Evidence

- `01_dns_record_created.png` — Shows the `Fileserver` Host (A) record.
- `02_Cname_alias_created.png` — Shows the `FileShare` CNAME alias.

---

## 2. Reverse DNS Lookup

Next, I configured reverse DNS.

Normally, DNS answers:

> **Hostname → IP address**

Reverse DNS does the opposite:

> **IP address → Hostname**

I created the reverse lookup zone:

`184.168.192.in-addr.arpa`

I then added PTR records for:

- `192.168.184.134` → `Fileserver.corp.local`
- `192.168.184.151` → `DESKTOP-IBEB2I4`

I tested the configuration using:

```cmd
nslookup 192.168.184.134
```

The lookup successfully returned the corresponding hostname.

### Evidence

- `03-reverse-dns-ptr-nslookup.png` — Shows the successful reverse lookup from Command Prompt.
- `04_reverse-lookup-created.png` — Shows the reverse lookup zone and PTR records in DNS Manager.

---

## 3. Testing DNS from Kali Linux

I wanted to make sure the DNS server was not only working from Windows itself.

To test cross-platform DNS resolution, I used the Kali Linux VM and queried the Windows Server DNS service directly:

```bash
dig @192.168.184.134 Fileserver.corp.local
```

The query returned:

- **Status:** `NOERROR`
- **Answer:** `192.168.184.134`

This confirmed that Kali Linux could successfully communicate with the Windows Server DNS service and resolve the `Fileserver.corp.local` hostname.

### Evidence

- `05_kali-dns-verification.png` — Shows the successful DNS query from Kali Linux.

---

# 4. DHCP Configuration

After testing DNS, I configured DHCP on `DC01`.

The purpose of DHCP is to automatically provide clients with the network information they need instead of configuring every machine manually.

I created a DHCP scope called:

**`Corp-LAN`**

The configured address pool was:

`192.168.184.150` – `192.168.184.200`

The DHCP server was configured on:

`192.168.184.134`

---

## DHCP Reservation

I also created a reservation for the Windows 10 client:

- **Reserved IP:** `192.168.184.150`
- **Client name:** `Windows10-Client`

The purpose of a reservation is to allow a particular device to consistently receive a specific IP address based on its network identity.

### Evidence

- `06-dhcp-scope.png` — Shows the active `Corp-LAN` DHCP scope and address range.
- `07-dhcp-reservation.png` — Shows the DHCP reservation for `192.168.184.150`.

---

# 5. Windows 10 DHCP Verification

Finally, I tested the DHCP configuration from the Windows 10 client.

I renewed the client's DHCP lease using:

```cmd
ipconfig /renew
```

I then checked the complete network configuration with:

```cmd
ipconfig /all
```

The client successfully received its network configuration.

### Final Client Results

| Setting | Result |
|---|---|
| IPv4 Address | `192.168.184.151` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.184.2` |
| DHCP Server | `192.168.184.134` |
| DNS Server | `192.168.184.134` |
| DNS Suffix | `corp.local` |

The important result here is that the Windows 10 client successfully received its network configuration through DHCP and was using `DC01` as its DHCP and DNS server.

> **Note:** The client received `192.168.184.151` during the final verification rather than the reserved `.150` address. The reservation configuration itself is documented separately in Screenshot 07. This documentation records the actual result observed during testing rather than changing the result to make it look cleaner.

### Evidence

- `08-dhcp-renewal-confirmation.png` — Shows the Windows 10 DHCP renewal and final `ipconfig /all` verification.

---

# 📸 Screenshot Reference Guide

| # | Screenshot | What It Shows |
|---|---|---|
| **01** | `01_dns_record_created.png` | `Fileserver` Host (A) record in `corp.local` |
| **02** | `02_Cname_alias_created.png` | `FileShare` CNAME pointing to `Fileserver.corp.local` |
| **03** | `03-reverse-dns-ptr-nslookup.png` | Successful reverse DNS lookup |
| **04** | `04_reverse-lookup-created.png` | Reverse Lookup Zone and PTR records |
| **05** | `05_kali-dns-verification.png` | Successful DNS query from Kali Linux |
| **06** | `06-dhcp-scope.png` | `Corp-LAN` DHCP scope and IP range |
| **07** | `07-dhcp-reservation.png` | DHCP reservation for `192.168.184.150` |
| **08** | `08-dhcp-renewal-confirmation.png` | Windows 10 DHCP renewal and network configuration |

---

# 🧠 What I Learned

This project gave me hands-on experience with several fundamental IT support and networking concepts:

### DNS

I learned how to:

- Create Host (A) records
- Create CNAME aliases
- Configure reverse DNS
- Create PTR records
- Test DNS using `nslookup`
- Test DNS from a Linux machine using `dig`

### DHCP

I learned how to:

- Create a DHCP scope
- Define an IP address pool
- Create DHCP reservations
- Renew a DHCP lease
- Verify DHCP information using `ipconfig /all`

### Troubleshooting

One of the most useful parts of the project was dealing with the fact that the lab did not always behave exactly as expected.

Instead of assuming everything was configured correctly, I verified the actual network information on the client and checked where the DHCP and DNS settings were coming from.

That is an important part of real IT support: **don't just assume the configuration is working—verify what the machine is actually receiving.**

---

# 🏁 Project Outcome

The completed lab demonstrates a working small-scale Windows network environment with:

- Windows Server 2019
- Active Directory
- DNS
- Forward DNS resolution
- Reverse DNS resolution
- DHCP
- DHCP reservations
- Windows 10 client configuration
- Cross-platform DNS testing with Kali Linux

The project also provided practical experience using Windows administration tools and command-line troubleshooting utilities.

This project is part of my **IT Support / Help Desk practical portfolio**, demonstrating hands-on experience rather than only theoretical knowledge.
