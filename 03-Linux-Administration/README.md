# Project 03 — Linux Administration

**Objective:** Configure, secure, manage, and troubleshoot a Linux system through practical day-to-day administrative tasks.

## Overview

This lab walks through setting up and administering a **Kali Linux 2026.3** virtual machine in VMware Workstation. Instead of focusing on theory, it documents the core tasks an IT systems administrator handles on a fresh Linux host: checking system baselines, provisioning user accounts, configuring SSH, setting up firewall rules, scheduling automated jobs, and auditing authentication logs.

### Core Tasks Covered
- System and kernel inspection
- User provisioning and `sudo` privilege management
- Network interface verification
- OpenSSH service configuration and testing
- Host firewall setup using UFW
- Task scheduling with Cron
- Authentication log investigation via `journalctl`
- Environment troubleshooting

---

## Lab Environment

| Component | Details |
| :--- | :--- |
| OS | Kali GNU/Linux Rolling 2026.3 |
| Kernel | `7.1.5+kali-amd64` |
| Hypervisor | VMware Workstation |
| Primary Interface | `eth0` (`192.168.184.135/24`) |
| Admin Account | `pauladmin` |

---

## Walkthrough & Evidence

### 1. System Information & Initial Network Check
Checked the OS release, hostname, kernel version, active user, and initial network state.

```bash
cat /etc/os-release
whoami
hostname
uname -a
ip addr
```

![System Information and Initial Network](./screenshots/01_System_Info_and_Initial_Network.png)

---

### 2. Sudo Privileges & Group Membership
Checked existing `sudo` rights and group memberships for the default `kali` user before creating a separate admin account.

```bash
sudo -l
groups
```

![Sudo Privileges and Groups](./screenshots/02_Sudo_Privileges_and_Groups.png)

---

### 3. Network Interface & IP Configuration
Re-checked `ip addr` once `eth0` initialized to confirm the link was up and had pulled an IPv4 address.

```bash
ip addr
```

![Network Interface IP Configuration](./screenshots/03_Network_Interface_IP_Configuration.png)

---

### 4. Starting the OpenSSH Service
Checked the SSH daemon status, started the service, and confirmed it was listening on port 22.

```bash
sudo systemctl status ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

![Start SSH Service](./screenshots/04_Start_SSH_Service.png)

---

### 5. Enabling SSH at Boot
Enabled the SSH service so it starts automatically after a reboot.

```bash
sudo systemctl enable ssh
sudo systemctl status ssh
```

![Enable SSH Service on Boot](./screenshots/05_Enable_SSH_Service_on_Boot.png)

---

### 6. Testing Local SSH Connectivity
Tested a local SSH login and verified the host key fingerprint.

```bash
ssh localhost
```

![Test Local SSH Connection](./screenshots/06_Test_Local_SSH_Connection_Kali.png)

---

### 7. Creating & Configuring an Administrative User
Created the `pauladmin` user with a home directory and bash shell, set a password, added the account to the `sudo` group, and confirmed root access.

```bash
sudo useradd -m -s /bin/bash pauladmin
sudo passwd pauladmin
sudo usermod -aG sudo pauladmin
groups pauladmin
su - pauladmin
sudo whoami
```

![Create and Configure Admin User](./screenshots/07_Create_and_Configure_Admin_User.png)

---

### 8. Testing SSH Access with the Administrative User
Verified that `pauladmin` could log in over SSH cleanly.

```bash
ssh pauladmin@localhost
whoami
hostname
```

![Test Local SSH Connection Pauladmin](./screenshots/08_Test_Local_SSH_Connection_Pauladmin.png)

---

### 9. Installing & Configuring UFW Firewall
Installed `ufw`, allowed SSH traffic before turning the firewall on so access wouldn't drop, enabled `ufw`, and checked the active rule set.

```bash
sudo apt update
sudo apt install ufw
ufw --version
sudo ufw allow ssh
sudo ufw enable
sudo ufw status
```

![Install and Configure UFW Firewall](./screenshots/09_Install_and_Configure_UFW_Firewall.png)

---

### 10. Automating Tasks with Cron
Added a cron entry to log the system date and time every minute and checked the log file to confirm it was running.

```bash
# Added via crontab -e:
* * * * * date >> /home/pauladmin/cron-test.log
```

```bash
cat /home/pauladmin/cron-test.log
```

![Configure and Verify Cron Job](./screenshots/10_Configure_and_Verify_Cron_Job.png)

---

### 11. Auditing SSH Authentication Logs
Checked SSH login history using `journalctl` after finding that `/var/log/auth.log` wasn't present on this build.

```bash
ls -l /var/log/auth.log
sudo journalctl -u ssh
sudo journalctl -u ssh | grep "Accepted"
sudo journalctl -u ssh | grep "Failed"
```

![Audit SSH Authentication Logs](./screenshots/11_Audit_SSH_Authentication_Logs.png)

---

## Troubleshooting Notes

| Issue | What Happened & How It Was Fixed |
| :--- | :--- |
| `eth0` wasn't showing an IP on first check | The interface was still initializing after boot. Re-running `ip addr` confirmed `eth0` was up with `192.168.184.135/24`. |
| `ufw` wasn't installed out of the box | Installed it via `apt`, allowed `ssh` first to avoid lockout, then enabled the firewall. |
| `/var/log/auth.log` didn't exist | This build uses `systemd` journal logging instead of legacy flat log files, so I queried `journalctl -u ssh` and filtered with `grep`. |

---

## Summary

This project covers the essential steps for getting a Linux host configured, secured, and verifiable from the command line:

- Verified OS, kernel, and network settings
- Set up a dedicated admin account with `sudo` rights
- Configured persistent SSH access and locked down traffic with UFW
- Automated a recurring task with Cron
- Audited SSH authentication events and resolved environment issues along the way
