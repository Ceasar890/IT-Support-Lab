# Project 06 — Active Directory & Group Policy Management

## Overview

For this project, I built and managed a small Windows domain environment to practice the kind of administration and troubleshooting tasks an IT support technician may handle in a real business environment.

I worked with a Windows Server 2019 domain controller and a Windows 10 domain-joined workstation. I created and linked Group Policy Objects (GPOs), applied security restrictions, deployed a shared network drive, and verified that the policies were actually reaching the client.

The goal was not just to configure the policies, but to **apply them, troubleshoot them when something went wrong, and verify the final result from the client side.**

---

## Lab Environment

| Component | Details |
| :--- | :--- |
| **Domain** | `corp.local` |
| **Domain Controller** | `DC01` — Windows Server 2019 |
| **Client** | `DESKTOP-IBEB2I4` — Windows 10 |
| **Test User** | `corp\testuser` |
| **Network Share** | `\\DC01\CompanyShare` |
| **Mapped Drive** | `H:` |

---

## What I Configured

### 1. Password Policy

I created a **Password Policy** GPO and configured password requirements for the domain environment.

The policy included:

- **Minimum password length:** 12 characters
- **Password complexity requirements:** Enabled

This gave me hands-on experience with configuring centralized password requirements through Group Policy.

### 2. Screen Lock Policy

I created a **Screen Lock Policy** to automatically protect an unattended workstation.

The policy was configured through:

`User Configuration → Policies → Administrative Templates → Control Panel → Personalization`

This enabled the screen saver and screen saver timeout so idle sessions lock automatically.

### 3. Removable Storage Restriction

I created a **Removable Storage Policy** to restrict access to removable storage devices.

The policy was configured to:

**All Removable Storage classes: Deny all access (Enabled)**

This provided practical experience with using Group Policy to reduce the risk of unauthorized data transfer and removable-media threats.

### 4. Group Policy Linking

The policies (`Password Policy` and `Screen Lock Policy`) were linked directly to the `corp.local` domain alongside the `Default Domain Policy` so that they could be processed by domain users and computers.

I also verified the link status (`Enabled`) and link order through **Group Policy Management**.

### 5. Network Share & Drive Mapping

I created a shared folder on the domain controller:

`\\DC01\CompanyShare`

I then created a **Drive Mapping GPO** using:

`User Configuration → Preferences → Windows Settings → Drive Maps`

The mapping was configured as:

- **Action:** Create
- **Location:** `\\DC01\CompanyShare`
- **Drive Letter:** `H:`

The purpose was to automatically give the domain user access to the shared resource without manually mapping the drive on the workstation.

---

## Troubleshooting & Verification

One of the most useful parts of this project was troubleshooting a real Group Policy connectivity problem.

The Windows 10 client initially had an incorrect network configuration and could not properly communicate with the domain controller. As a result, Group Policy was not refreshing correctly.

I investigated the problem by checking the client's network configuration and connectivity to the domain controller.

After correcting the client's static network configuration, I verified connectivity to `DC01` and successfully refreshed Group Policy.

I used:

```cmd
gpupdate /force
```

to force the workstation to retrieve the latest policies.

I then used:

```cmd
gpresult /r
```

to verify which GPOs were actually being applied to `corp\testuser`. Both **Screen Lock Policy** and **Drive Mapping GPO** appeared under **Applied Group Policy Objects**, confirming that the policies were reaching the client.

Finally, I checked **File Explorer** and confirmed that the `H:` drive was successfully mapped to:

`\\DC01\CompanyShare`

---

## Evidence

The project includes screenshots documenting the configuration and verification process in chronological order.

### Screenshot 01 — Password Policy

![Password Policy GPO Settings](01-password-policy.png)

Shows the configured domain password policy enforcing a 12-character minimum length and complexity requirements.

### Screenshot 02 — Screen Lock Policy

![Screen Lock Policy Settings](02-Screen-lock-policy.png)

Shows the configured screen-lock and screen-saver timeout settings under User Configuration.

### Screenshot 03 — GPO Linked to Domain

![GPO Linked to Domain](03_gpo_linkage.png)

Shows the Group Policy objects linked to `corp.local` and their active link order in Group Policy Management.

### Screenshot 04 — Group Policy Update

![Group Policy Update Success](04_gpupdate_success.png)

Shows the successful `gpupdate /force` operation completing both computer and user policy updates on the Windows 10 client.

### Screenshot 05 — Applied Group Policies

![Applied Group Policies via gpresult](05_gpresult_screen_lock_policy.png)

Shows the `gpresult /r` RSoP logging output confirming `Screen Lock Policy` is applied to `corp\testuser`.

### Screenshot 06 — Removable Storage Restriction

![Removable Storage Restriction Policy](06-removable-storage-restriction.png)

Shows the removable-storage restriction policy denying access to all removable storage classes.

### Screenshot 07 — Mapped H: Drive

![Mapped H: Drive in File Explorer](07-mapped-H-drive.png)

Shows the `H:` drive (`CompanyShare (\\DC01)`) successfully mapped and visible in Windows 10 File Explorer alongside `gpresult /r` confirming `Drive Mapping GPO`.

### Screenshot 08 — Drive Mapping GPO

![Drive Mapping GPO Settings](08-drive-mapping-gpo-settings.png)

Shows the Drive Mapping GPO configured under Group Policy Preferences to map `\\DC01\CompanyShare` to `H:`.

---

## Skills Practiced

Through this project, I gained hands-on practice with:

- Active Directory domain administration
- Group Policy Management & Group Policy Preferences
- Password, screen-lock, and removable-storage security policies
- SMB network shares and automated drive mapping
- Windows client administration and GPO linking
- Group Policy diagnostics (`gpupdate /force`, `gpresult /r`, RSoP)
- Domain connectivity and DNS/network troubleshooting

---

## What I Learned

The biggest lesson from this project was that configuring a Group Policy is only part of the job.

A policy can be configured correctly on the server and still fail to reach the user if the client cannot communicate properly with the domain controller.

Working through the connectivity problem helped me understand the relationship between:

**Network connectivity → Domain communication → Group Policy processing → Client configuration**

I also learned the importance of verifying changes instead of assuming they worked. Using `gpupdate /force`, `gpresult /r`, and File Explorer allowed me to confirm the configuration from the client side.

---

## Final Result

The project was successfully completed.

The domain policies were configured and linked, Group Policy was successfully refreshed on the Windows 10 client, the expected GPOs were verified as applied, and the `H:` network drive was successfully deployed from the domain controller.

This project gave me practical experience with a complete **configure → troubleshoot → verify** workflow that I can apply to real IT support situations.
