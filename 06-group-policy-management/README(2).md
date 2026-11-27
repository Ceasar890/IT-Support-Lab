# Active Directory & Group Policy Management Lab

## Overview

In this project, I built and administered a Windows Server Active Directory domain environment (`corp.local`) using VMware Workstation to simulate day-to-day enterprise system administration and IT support workflows. 

Working across a Windows Server 2019 Domain Controller (`DC01`) and a domain-joined Windows 10 workstation (`DESKTOP-IBEB2I4`), I engineered security-hardening Group Policy Objects (GPOs), deployed automated network storage via Group Policy Preferences, resolved client-to-DC connectivity hurdles, and verified policy inheritance from the end-user perspective.

---

## Lab Environment

| Component | Configuration Details |
| :--- | :--- |
| **Virtualization Platform** | VMware Workstation |
| **Active Directory Domain** | `corp.local` |
| **Domain Controller** | `DC01` — Windows Server 2019 Standard Evaluation |
| **Client Workstation** | `DESKTOP-IBEB2I4` — Windows 10 (OS Build `10.0.19045`) |
| **Domain Test Account** | `corp\testuser` (`CN=Test User,CN=Users,DC=corp,DC=local`) |
| **Shared Network Folder** | `\\DC01\CompanyShare` (Mapped to `H:`) |

---

## Implementation Walkthrough

### 1. Domain Password Policy Enforcement
To establish baseline credential security across the domain, I configured password rules under **Account Policies → Password Policy**:
* **Minimum password length:** `12 characters`
* **Password must meet complexity requirements:** `Enabled`

![Password Policy GPO Settings](./01-password-policy.png)

---

### 2. Unattended Workstation Screen Lock Policy
To prevent unauthorized physical access to idle workstations, I created the **Screen Lock Policy** GPO under **User Configuration → Policies → Administrative Templates → Control Panel → Personalization**:
* **Enable screen saver:** `Enabled`
* **Screen saver timeout:** `Enabled`

![Screen Lock Policy GPO Settings](./02-Screen-lock-policy.png)

---

### 3. Group Policy Linking & Precedence
Using the **Group Policy Management Console (GPMC)**, I linked the custom GPOs directly to the `corp.local` domain root and verified link status and processing order alongside the `Default Domain Policy`:
1. `Default Domain Policy` (Link Order 1 — Enabled)
2. `Password Policy` (Link Order 2 — Enabled)
3. `Screen Lock Policy` (Link Order 3 — Enabled)

![Group Policy Linkage on corp.local](./03_gpo_linkage.png)

---

### 4. Client Policy Refresh & Resultant Set of Policy (RSoP) Verification
After ensuring proper network and DNS communication between the Windows 10 workstation and `DC01`, I forced an immediate policy refresh from the command prompt:

```cmd
gpupdate /force
```

![Successful gpupdate /force Output](./04_gpupdate_success.png)

Next, I logged in as `corp\testuser` on `DESKTOP-IBEB2I4` and ran `gpresult /r` to audit the Resultant Set of Policy (RSoP). The output confirmed that Group Policy was pulled directly from `DC01.corp.local` and that the **Screen Lock Policy** was actively applied:

```cmd
gpresult /r
```

![RSoP Verification via gpresult](./05_gpresult_screen_lock_policy.png)

---

### 5. Removable Storage Restriction (Data Loss Prevention)
To reduce the risk of malware introduction and unauthorized data transfers via USB drives or external media, I configured a removable-storage restriction policy under **Administrative Templates → System → Removable Storage Access**:
* **All Removable Storage classes: Deny all access:** `Enabled`

![Removable Storage Restriction Policy](./06-removable-storage-restriction.png)

---

### 6. Automated Network Drive Mapping via Group Policy Preferences
To provide users with seamless access to centralized department files without manual setup, I created a shared directory (`\\DC01\CompanyShare`) and deployed a **Drive Mapping GPO** under **User Configuration → Preferences → Windows Settings → Drive Maps**:
* **Action:** `Create`
* **Location:** `\\DC01\CompanyShare`
* **Drive Letter:** `H:`

![Drive Mapping GPO Properties](./08-drive-mapping-gpo-settings.png)

Once the policy refreshed on `DESKTOP-IBEB2I4`, File Explorer confirmed that **`CompanyShare (\\DC01) (H:)`** mounted automatically for `corp\testuser`:

![Mapped H: Drive in Windows 10 File Explorer](./07-mapped-H-drive.png)

---

## Troubleshooting & Key Takeaways

* **DNS & Network Reachability Drive GPO Success:** During client testing, Group Policy updates initially stalled until the Windows 10 workstation's static IP and DNS settings were properly aligned to resolve `DC01.corp.local`. Verifying network reachability first is essential before diagnosing GPO settings.
* **Policies vs. Preferences:** Traditional GPO Administrative Templates enforce strict system restrictions (such as locking down removable storage), while Group Policy Preferences allow flexible user-environment provisioning like automatic `H:` drive mapping.
* **End-to-End Verification:** Combining `gpupdate /force`, `gpresult /r`, and direct UI checks in File Explorer ensures policies are not only configured on the server, but actually delivered to the end user.

---

## Core Skills Demonstrated

* **Active Directory & Group Policy:** Domain GPO creation, link precedence, security hardening, and Group Policy Preferences (GPP)
* **Endpoint Security & DLP:** Password complexity enforcement, session screen-lock controls, and removable-media access restrictions
* **Storage Provisioning:** SMB file share configuration and automated network drive mapping
* **Diagnostics & Troubleshooting:** Windows CLI (`gpupdate /force`, `gpresult /r`), RSoP analysis, and client-to-DC DNS/network troubleshooting
