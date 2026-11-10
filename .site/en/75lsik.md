
> **Goal of this section:**
> Be able to create and manage users on Linux confidently with `useradd`, understand the difference between it and `adduser`, and know how to Verify every step before moving on to the next one.




## Table of Contents

- [Pre-Flight Checklist](#pre-flight-checklist)
- [useradd vs adduser](#useradd-vs-adduser)
- [What Happens When You Run useradd?](#what-happens-when-you-run-useradd)
- [Syntax](#syntax)
- [Step-by-Step Usage](#step-by-step-usage)
- [Verify User](#verify-user)
- [Quick Reference](#quick-reference)
- [Post-Lab Checklist](#post-lab-checklist)
- [Security Notes (SOC / Blue Team)](#security-notes-soc--blue-team)

---

## Pre-Flight Checklist

- [ ] You have `root` or `sudo` privileges on the machine
- [ ] The machine you'll practice on is a test environment (VM/Lab), not a Production server
- [ ] You know the Username you'll create in advance
- [ ] You have a Terminal open and ready

---

## useradd vs adduser

| Aspect | `useradd` | `adduser` |
|---|---|---|
| Type | Native binary | Perl script |
| Interaction Level | Low-level, less interactive | User-friendly, interactive |
| Behavior | Requires explicit flags for most settings | Asks for password, full name, etc. automatically |
| Relationship | - | Uses `useradd` internally |
| Recommended Use | Servers and scripts | Manual/interactive desktop use |

> [!IMPORTANT]
> For professional work and in exams, the preference is always `useradd`, because it's the Native command, and understanding it gives you full control over every detail that happens when a user is created.

---

## What Happens When You Run useradd?

The `useradd` command directly modifies these system files:

| File | Purpose |
|---|---|
| `/etc/passwd` | stores basic user information |
| `/etc/shadow` | stores encrypted passwords |
| `/etc/group` | stores primary group info |
| `/etc/gshadow` | stores group permissions |
| `/home/username` | home directory (if created) |

---

## Syntax

```bash
useradd [options] username
```

**Requirements**: You must run it as `root` or using `sudo`.

---

## Step-by-Step Usage

### Step 1: Create a Basic User

```bash
sudo useradd test_user
```

- Creates a new user
- No password or home directory by default
- User cannot log in until a password is set

**Set password**:

```bash
sudo passwd test_user
```

> [!CHECKPOINT]
> Check: Run `id test_user` - it should return the new user's UID and GID, which confirms the creation succeeded before you continue.

### Step 2: Specify Home Directory

```bash
sudo useradd -d /home/test_user test_user
```

- Sets a custom home directory
- Does not create it automatically unless `-m` is used

**Recommended**:

```bash
sudo useradd -m -d /home/test_user test_user
```

> [!WARNING]
> If you forget `-m`, the command will only set the path **without actually creating the directory**. If the user tries to Log in, they may run into problems because the Home Directory doesn't exist at all.

### Step 3: Specify User ID (UID)

```bash
sudo useradd -u 1234 test_user
```

- Assigns a specific UID
- Useful for syncing across servers, LDAP, or NFS

> [!NOTE]
> The UID must be Unique, and it's usually greater than 1000 for regular users (lower numbers are usually reserved for system accounts).

### Step 4: Specify Group ID (GID)

```bash
sudo useradd -g 1000 test_user
```

It adds the user to an existing group using its GID.

### Step 5: Create User Without Home Directory

```bash
sudo useradd -M test_user
```

- Useful for service accounts or restricted users
- Enhances system security

### Step 6: Set Account Expiry Date

```bash
sudo useradd -e 2020-05-30 test_user
```

- User cannot log in after this date
- Useful for temporary accounts or trainees

### Step 7: Add Comment / Description

```bash
sudo useradd -c "This is a test user" test_user
```

- Adds a description in `/etc/passwd`
- Typically includes full name, job title, or employee ID

### Step 8: Change Login Shell

```bash
sudo useradd -s /bin/sh test_user
```

- Assigns a specific login shell (default is `/bin/bash`)
- Useful for restricted shells or service users

### Step 9: Set Unencrypted Password (⚠️ Not Recommended)

```bash
sudo useradd -p test_password test_user
```

> [!WARNING]
> **Password is plain text**. Use `sudo passwd test_user` instead for security.

### Step 10: Display Help

```bash
sudo useradd --help
```

Or:

```bash
man useradd
```

---

## Verify User

After any creation or modification step, always verify using:

```bash
id test_user
```

```bash
getent passwd test_user
```

> [!CHECKPOINT]
> Final check: Compare the output of `getent passwd test_user` with the settings you intended to apply (Home Directory, Shell, UID/GID). If anything doesn't match, go back to the step you performed and double-check the Flags you used.

---

## Quick Reference

| Flag | Purpose |
|---|---|
| `useradd` | Create a user |
| `passwd` | Always set password after creating a user |
| `-m` | Create home directory |
| `-u` | Set UID |
| `-g` | Set primary GID |
| `-s` | Set shell |
| `-e` | Set expiry date |
| `-M` | Do not create home directory |
| `-c` | Add comment/description |
| `-d` | Specify home directory path |

---

## Post-Lab Checklist

- [ ] The user was created successfully and appears in `getent passwd`
- [ ] The password was set using `passwd` (not via `-p`)
- [ ] The Home Directory was actually created if it was required (`-m` was used)
- [ ] The Shell and UID/GID match what was required
- [ ] If the user is temporary, the Expiry date was set correctly
- [ ] If it was only a test account, delete it when you're done using `userdel -r test_user`

---

## Security Notes (SOC / Blue Team)

> [!IMPORTANT]
> The `useradd` command modifies very sensitive files such as `/etc/passwd` and `/etc/shadow`, and any unexpected use of it on a Production server must be raised as an immediate Alert.

| Risk | Description | MITRE ATT&CK Reference |
|---|---|---|
| Unauthorized Account Creation | An attacker who has compromised a machine creates a new user with a normal UID to use as a persistent Backdoor | T1136.001 - Create Account: Local Account |
| Plaintext Password via `-p` | Using `-p` records the password in a way that can be spotted in the Shell History or the Logs | T1552.003 - Unsecured Credentials: Bash History |
| UID 0 Assignment (Privilege Escalation) | Assigning UID = 0 to a new user gives it full root privileges without its name being "root" | T1078.003 - Valid Accounts: Local Accounts |
| Hidden Service Accounts | Creating accounts with `-M` (no Home Directory) and an unusual Shell to reduce the visible footprint and make hiding easier | T1136 - Create Account |

> [!TIP]
> If you're investigating a Linux machine, run this command to look for any user with UID = 0 other than `root` itself (a very serious indicator of Privilege Escalation):
> ```bash
> awk -F: '$3 == 0 {print $1}' /etc/passwd
> ```
> If it returns any name other than `root`, it deserves immediate investigation.

### Detection & Best Practices

- Monitor any execution of the `useradd` command on Production servers through Audit Logs (`auditd` or `/var/log/auth.log`)
- Review `/etc/passwd` regularly to make sure there are no undocumented users or duplicate UID = 0 entries
- Avoid using `-p` entirely in any real environment, and rely only on the interactive `passwd`
- Enable immediate Alerts on any modification to the `/etc/passwd` or `/etc/shadow` files outside the usual maintenance window
