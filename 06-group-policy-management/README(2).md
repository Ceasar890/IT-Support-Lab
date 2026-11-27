# Project 06 — Active Directory & Group Policy Management

## Overview

For this project, I built and managed a small Windows domain environment to practice the kind of administration and troubleshooting tasks an IT support technician may handle in a real business environment.

I worked with a Windows Server 2019 domain controller (`DC01`) and a Windows 10 domain-joined workstation (`DESKTOP-IBEB2I4`). I created and linked Group Policy Objects (GPOs), applied security restrictions, deployed a shared network drive, and verified that the policies were actually reaching the client.

The goal was not just to configure the policies, but to **apply them, troubleshoot them when something went wrong, and verify the final result from the client side.**

---

## Lab Environment

| Component | Details |
| :--- | :--- |
| **Domain** | `corp.local` |
| **Domain Controller** | `DC01` — Windows Server 2019 Standard |
| **Client Workstation** | `DESKTOP-IBEB2I4` — Windows 10 |
| **Test User** | `corp\testuser` |
| **Network Share** | `\\DC01\CompanyShare` |
| **Mapped Drive** | `H:` |

---

## What I Configured

### 1. Password Policy

I created a **Password Policy** GPO and configured password requirements for the domain environment under `Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy`.

The policy enforced:

- **Minimum password length:** 12 characters
- **Password complexity requirements:** Enabled

This gave me hands-on experience with configuring centralized password requirements through Group Policy.

### 2. Screen Lock Policy

I created a **Screen Lock Policy** to automatically protect unattended workstations.

The policy was configured through:

`User Configuration → Policies → Administrative Templates → Control Panel → Personalization`

Key settings enabled:

- **Enable screen saver:** Enabled
- **Screen saver timeout:** Enabled

This helped me understand how user-based Group Policy settings can be used to enforce workstation security.

### 3. Removable Storage Restriction

I created a **Removable Storage Policy** to restrict access to external storage media.

The policy was configured to:

- **All Removable Storage classes: Deny all access:** Enabled

This provided practical experience with using Group Policy to reduce the risk of unauthorized data transfer and removable-media threats.

### 4. Group Policy Linking

The custom GPOs (`Password Policy` and `Screen Lock Policy`) were linked directly to the `corp.local` domain root alongside the `Default Domain Policy` so they could be processed by domain users and computers.

I verified the link order, enforcement status, and enabled state through **Group Policy Management**.

### 5. Network Share & Drive Mapping

I created a shared folder on the domain controller:

`\\DC01\CompanyShare`

I then created a **Drive Mapping GPO** using Group Policy Preferences:

`User Configuration → Preferences → Windows Settings → Drive Maps`

The mapping was configured as:

- **Action:** Create
- **Location:** `\\DC01\CompanyShare`
- **Drive Letter:** `H:`

The purpose was to automatically give domain users access to the shared company resource without manually mapping the drive on each workstation.

---

## Troubleshooting & Verification

One of the most useful parts of this project was troubleshooting a real Group Policy connectivity problem.

The Windows 10 client initially had an incorrect static network configuration and could not properly communicate with the domain controller. As a result, Group Policy was not refreshing correctly.

I investigated the problem by checking the client's IP and DNS settings and testing connectivity to `DC01`.

After correcting the client's static network configuration, I verified connectivity to `DC01` and forced an immediate Group Policy refresh:

```cmd
gpupdate /force
```

Both Computer Policy and User Policy updates completed successfully.

I then ran Resultant Set of Policy (RSoP) logging from the command line:

```cmd
gpresult /r
```

This confirmed that `corp\testuser` on `DESKTOP-IBEB2I4` was receiving policies from `DC01.corp.local` and verified that **Screen Lock Policy** and **Drive Mapping GPO** were listed under **Applied Group Policy Objects**.

Finally, I opened **File Explorer** on the Windows 10 client and confirmed that `CompanyShare (\\DC01) (H:)` was successfully mapped and accessible.

---

## Visual Walkthrough & Evidence

Each step of the configuration, troubleshooting, and client verification process is documented below:

### 01 — Password Policy Configuration

![Password Policy GPO Settings](./01-password-policy.png)

Shows the domain **Password Policy** configured with a 12-character minimum password length and complexity requirements enabled.

---

### 02 — Screen Lock Policy Configuration

![Screen Lock Policy Settings](./02-Screen-lock-policy.png)

Shows the **Screen Lock Policy** user settings enabling the screen saver and screen saver timeout.

---

### 03 — Group Policy Domain Linkage

![GPOs Linked to Domain](./03_gpo_linkage.png)

Shows the **Group Policy Management Console (GPMC)** with `Default Domain Policy`, `Password Policy`, and `Screen Lock Policy` linked and enabled on `corp.local`.

---

### 04 — Forced Group Policy Refresh

![Successful gpupdate /force](./04_gpupdate_success.png)

Shows `gpupdate /force` completing both Computer and User Policy updates on the Windows 10 workstation (`DESKTOP-IBEB2I4`).

---

### 05 — Applied Group Policy Verification

![gpresult /r Applied Policies](./05_gpresult_screen_lock_policy.png)

Shows the `gpresult /r` RSoP logging output confirming policy retrieval from `DC01.corp.local` and verifying `Screen Lock Policy` under **Applied Group Policy Objects** for `corp\testuser`.

---

### 06 — Removable Storage Restriction Policy

![Removable Storage Restriction Settings](./06-removable-storage-restriction.png)

Shows the administrative template policy **All Removable Storage classes: Deny all access** set to **Enabled**.

---

### 07 — Mapped Network Drive on Windows 10 Client

![Mapped H: Drive in File Explorer](./07-mapped-H-drive.png)

Shows Windows 10 File Explorer confirming `CompanyShare (\\DC01) (H:)` is mapped and ready for use.

---

### 08 — Drive Mapping GPO Configuration

![Drive Mapping GPO Settings](./08-drive-mapping-gpo-settings.png)

Shows Group Policy Preferences (`H: Properties`) configured with **Action: Create**, **Location:** `\\DC01\CompanyShare`, and **Drive Letter:** `H:`.

---

## Skills Practiced

- **Active Directory & Group Policy:** Domain administration, GPO creation, domain-level GPO linking, Group Policy Preferences (GPP)
- **Endpoint Security Hardening:** Password length/complexity policies, workstation screen-lock enforcement, removable storage/USB access restrictions
- **Storage & Resource Provisioning:** SMB file share creation and automated user network drive mapping
- **Client Diagnostics & Troubleshooting:** `gpupdate /force`, `gpresult /r` (RSoP), static IP/DNS configuration, domain controller connectivity verification

---

## What I Learned

The biggest lesson from this project was that configuring a Group Policy is only part of the job.

A policy can be configured correctly on the server and still fail to reach the user if the client cannot communicate properly with the domain controller.

Working through the connectivity issue reinforced the end-to-end dependency chain:

**Network & DNS Connectivity → Domain Communication → Group Policy Processing → Client Configuration**

It also emphasized the importance of verifying changes directly from the client side using `gpupdate /force`, `gpresult /r`, and File Explorer rather than assuming server-side changes applied automatically.

---

## Final Result

The project was successfully completed. Domain security policies were configured and linked, Group Policy refreshed cleanly on the Windows 10 client after resolving network connectivity, the expected GPOs were verified via `gpresult /r`, and the `H:` network drive was automatically mapped to `\\DC01\CompanyShare`.

This lab demonstrated a complete **configure → troubleshoot → verify** workflow applicable to real-world IT support and systems administration roles.
