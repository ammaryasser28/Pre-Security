> **Goal of this section:**  
> You'll be able to delete users from Linux safely and the right way, tell the difference between a regular deletion and a full deletion (`-r`), and know when to use `-f` with extreme caution.


## Table of Contents

- [Pre-Flight Checklist](#pre-flight-checklist)
- [What is userdel?](#what-is-userdel)
- [Syntax](#syntax)
- [Common Options](#common-options)
- [Step-by-Step Usage](#step-by-step-usage)
- [Notes for System Administrators](#notes-for-system-administrators)
- [Quick Reference](#quick-reference)
- [Post-Lab Checklist](#post-lab-checklist)
- [Security Notes (SOC / Blue Team)](#security-notes-soc--blue-team)

---

## Pre-Flight Checklist

- [ ] You have `root` or `sudo` privileges on the machine
- [ ] The machine you'll practice on is a test environment (VM/Lab), not a production server
- [ ] You're 100% sure of the username you're deleting (there's no going back after deletion!)
- [ ] You've made a **Backup** of any important files inside the user's home directory if you need them
- [ ] You've confirmed the user has no sensitive processes or services currently running

> [!WARNING]
> Deletion with `userdel` is **irreversible**, especially with `-r`. Double-check the username before executing.

---

## What is userdel?

- Deletes a user account from the system
- Removes references to the username in system files like `/etc/passwd`, `/etc/shadow`, `/etc/group`, `/etc/gshadow`
- Can remove home directories and mail spools associated with the user
- Offers various options for force deletion, SELinux cleanup, or working in chroot environments

---

## Syntax

```bash
userdel [options] username
```

- `options`: Modify the behavior of deletion
- `username`: Name of the user to delete
- Must be run as **root** or with **sudo**

---

## Common Options

| Option | Purpose |
|---|---|
| (none) | Delete only the user account, without touching the home directory or mail spool |
| `-f` | Forcefully delete a user, even if they are currently logged in |
| `-r` | Delete user along with their home directory and mail spool |
| `-h` | Display quick help message |
| `-R /path/to/chroot` | Apply changes within a specific chroot directory |
| `-Z` | Remove SELinux user mapping |
| `--help` | Show general syntax and all options |

---

## Step-by-Step Usage

### Step 1: Delete a User

```bash
sudo userdel username
```

- Deletes the user account from the system
- Replace `username` with the actual account name
- User cannot be deleted without administrative privileges (`sudo` required)

> [!CHECKPOINT]
> Verify: run `getent passwd username` after the deletion - it should return nothing, which confirms the account was removed from `/etc/passwd`.

### Step 2: Forcefully Delete a User

```bash
sudo userdel -f neuser
```

- Forces deletion even if the user is logged in
- Removes home directory and mail spool regardless of ownership

> [!WARNING]
> Using `-f` can cause an **inconsistent system state** if the user had running processes at the time of deletion. Use it with extreme caution and only when it's truly necessary.

### Step 3: Delete User with Home Directory and Mail Spool

```bash
sudo userdel -r newuser2
```

- Deletes user account and all files in their home directory
- Mail spool is also deleted
- Files on other file systems must be removed manually

> [!IMPORTANT]
> `-r` deletes only the files inside the **Home Directory** and the **Mail Spool**. Any of the user's files located elsewhere on the system (such as `/opt` or project folders outside the home) must be deleted manually by you.

### Step 4: Display Help Message

```bash
userdel -h
```

It shows you a quick reference for the syntax and the available options.

### Step 5: Apply Changes in a CHROOT_DIR

```bash
sudo userdel -R /path/to/chroot username
```

- Uses configuration files from the specified chroot directory
- Useful in containers, recovery environments, or isolated systems

### Step 6: Remove SELinux User Mapping

```bash
sudo userdel -Z newuser2
```

- Deletes SELinux mappings associated with the user account
- Ensures complete removal from SELinux-enabled systems

> [!NOTE]
> If the system uses **SELinux**, you must use `-Z` to ensure the user's mapping is removed completely. If you forget it, a residual mapping for the user may remain in the SELinux configuration even after the account itself is deleted.

### Step 7: General Help Option

```bash
userdel --help
```

It shows you the general syntax and all the available options - useful if you forget any flag or usage.

---

## Notes for System Administrators

- Deleting user accounts is critical for security and system cleanup
- Use `-r` for full cleanup to remove home directories and mail spools
- Use `-f` cautiously to remove users who may be logged in
- SELinux users should also use `-Z` to remove security mappings

---

## Quick Reference

| Command | Purpose |
|---|---|
| `sudo userdel username` | Delete user account only |
| `sudo userdel -f username` | Force delete (even if logged in) |
| `sudo userdel -r username` | Delete user + home directory + mail spool |
| `sudo userdel -R /path username` | Apply deletion within a chroot directory |
| `sudo userdel -Z username` | Remove SELinux user mapping |
| `userdel -h` / `userdel --help` | Show help |

---

## Post-Lab Checklist

- [ ] The user has disappeared from `getent passwd username`
- [ ] The home directory was removed if you used `-r` (confirm with `ls /home/`)
- [ ] If you used `-Z`, confirm the SELinux mapping was removed (if the system supports SELinux)
- [ ] You've reviewed the rest of the system to make sure there are no leftover files for the user on file systems other than the home
- [ ] If the user was part of shared groups, confirm the remaining members aren't affected

---

## Security Notes (SOC / Blue Team)

> [!IMPORTANT]
> `userdel` isn't just an ordinary administrative tool - in a security context, deleting users (or **not** deleting them at the right time) is a core part of the account lifecycle management cycle, and any failing there leaves open doors.

| Risk | Description | MITRE ATT&CK Reference |
|---|---|---|
| Orphaned/Stale Accounts | Failing to delete the accounts of employees who left the company or temporary contractors leaves valid accounts that can be exploited | T1078 - Valid Accounts |
| Anti-Forensics via userdel | After using a temporary account in an attack, an attacker can delete it with `userdel -r` to wipe their traces and make tracking harder | T1070 - Indicator Removal |
| Incomplete Cleanup (Missing -Z) | Forgetting `-Z` in SELinux environments leaves residual mappings that can be exploited or cause confusion in security policies | T1548 - Abuse Elevation Control Mechanism |
| Force Deletion of Active Sessions | Using `-f` without justification can suddenly cut off a legitimate session, which can be used as a form of internal denial of service if done maliciously | T1531 - Account Access Removal |

> [!WARNING]
> **Account Access Removal (T1531)** is a known technique that attackers use not just to cover their traces, but also as a destructive tactic - such as deleting legitimate users' accounts to block their access during a ransomware attack, for example. Any unexpected use of `userdel` on active accounts should be raised immediately as a critical alert.

### Detection & Best Practices

- Monitor any execution of `userdel` on servers through **Audit Logs** (`auditd`, `/var/log/auth.log`), especially the use of `-f` or `-r`
- Tie user deletion to a formal, documented offboarding process linked to Human Resources, not random manual commands
- Review `/etc/passwd` and `/etc/group` periodically to detect any stale accounts that should have been deleted long ago
- Make sure `-Z` is applied in every deletion on systems that use SELinux, as part of the standard procedure

> [!TIP]
> If you notice in the logs that a certain account was created with `useradd` and then, a very short time later (minutes or hours), deleted with `userdel -r`, that's a highly suspicious pattern - it could be an attacker who used a temporary account to execute something and then deleted it to erase their traces. Review the logs related to this account's activity during that short period immediately.
