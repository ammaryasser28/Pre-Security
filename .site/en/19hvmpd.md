> **Goal of this section:**
> Understand the critical difference between a Login Shell and a Non-Login Shell when using `su`, and know how to use it safely, whether to become root, to test another user's permissions, or to run just a single command with different privileges.


## Table of Contents

- [Pre-Flight Checklist](#pre-flight-checklist)
- [Key Uses of su](#key-uses-of-su)
- [Syntax](#syntax)
- [Login vs Non-Login Shell](#login-vs-non-login-shell)
- [Common Options](#common-options)
- [Step-by-Step Usage](#step-by-step-usage)
- [Tips](#tips)
- [Post-Lab Checklist](#post-lab-checklist)
- [Security Notes (SOC / Blue Team)](#security-notes-soc--blue-team)

---

## Pre-Flight Checklist

- [ ] You know the difference between `su -` and `su` before running any command
- [ ] You have the target user's (or root's) password ready
- [ ] The machine you'll practice on is a test environment (VM/Lab)
- [ ] You're sure of the exact username you'll switch to

---

## Key Uses of su

- **Become root**: Gain administrative privileges to perform system-wide tasks
- **Test permissions**: Switch to a regular user account (e.g., service accounts like `www-data`)
- **Run commands as another user**: Execute a single command with the privileges of another user

---

## Syntax

```bash
su [OPTION] [USER]
```

- `OPTION` → command-line options to control behavior
- `USER` → the username you want to switch to
- Must be run as **root** or with **sudo**

---

## Login vs Non-Login Shell

The most important concept in this Runbook is the difference between a **Login Shell** and a **Non-Login Shell**.

### `su - [username]` (Login Shell – Recommended)

- The dash `-` is short for `--login`
- Loads the full environment of the target user (`.bashrc`, `.profile`)
- Sets the PATH and switches to the user's home directory

**Examples**:

```bash
# Become root
su -

# Become a regular user
su - user
```

### `su [username]` (Non-Login Shell)

- Switches user ID only
- Keeps your current environment and PATH
- Does not change the current directory

> [!WARNING]
> **Risk**: Using a Non-Login Shell as root can lead you to create files in the wrong place or run commands with the wrong PATH - because your old Environment (the original user's) is the one still in effect, not root's.

> [!IMPORTANT]
> **Recommendation**: Always use `su -` unless you have a specific, genuine reason to use a Non-Login Shell.

---

## Common Options

| Command | Behavior |
|---|---|
| `su -` | Switch to root with full login shell (**Recommended**) |
| `su` | Switch to root but keep your current environment (**Not recommended**) |
| `su - [user]` | Switch to `[user]` with login shell (**Recommended**) |
| `su [user]` | Switch to `[user]` but keep your environment |
| `su -c "command"` | Execute a single command as root (with root's environment) and exit |
| `su -p [user]` | Preserve your current environment while switching to `[user]` |
| `-l` or `--login` | Full flag for login shell (`-` equivalent) |
| `-s /bin/sh` | Specify a different login shell |

---

## Step-by-Step Usage

### Step 1: Switch to Root

```bash
sudo su
```

- Enter the root password when prompted
- Gain superuser privileges until you exit the session

> [!CHECKPOINT]
> Check: Run `whoami` after switching - it should return `root`. If it returns the old username, something is wrong.

### Step 2: Switch to Another User

```bash
su username
```

- Enter the target user's password
- Example:

```bash
su shivansh260
```

It switches you to the user `shivansh260` and their Home Directory.

### Step 3: Execute a Command as Another User

```bash
sudo su -c "command" -s /bin/bash username
```

- Runs `command` as `[username]` without switching fully
- `-s /bin/bash` specifies the shell to use

**Example**:

```bash
sudo su -c "echo hello" -s /bin/bash shivansh
```

> [!TIP]
> This command is very useful in automated Scripts when you need to run just one command with a specific user's privileges, without opening a full Session and closing it manually.

### Step 4: Preserve Environment Variables

```bash
sudo su -
```

- Use `-` or `--login` to load the target user's full environment
- Ensures PATH and environment variables are correctly set

### Step 5: Simulate a Full Login Shell

```bash
su -l username
```

- Fully loads the target user's environment as if logging in normally
- Recommended for testing user-specific settings and scripts

> [!CHECKPOINT]
> Check: Run `echo $PATH` and `pwd` after any switch - if you used `-` or `-l`, you should be in the new user's Home Directory with their PATH in effect. If you didn't use `-`, you'll find you're still in the same place with the old PATH.

---

## Tips

1. Always prefer `su -` for safe environment switching
2. Use `su -c "command"` to run single commands as another user
3. Non-login shells should only be used if you understand the risks
4. Useful for system admins, service accounts, and troubleshooting permissions

---

## Post-Lab Checklist

- [ ] You confirmed the current user's identity after every switch using `whoami`
- [ ] You used `su -`, not plain `su`, unless there was a clear reason
- [ ] If you ran a command with another user's privileges, you verified the result before continuing
- [ ] You exited any open root Session using `exit` after finishing your work
- [ ] If you were testing permissions, you returned to your original account at the end

---

## Security Notes (SOC / Blue Team)

> [!IMPORTANT]
> The `su` command is one of the commands that must be monitored most carefully in any environment, because it represents a direct switch to higher privileges (a Privilege Escalation Point) - whether it's legitimate use by an Admin or abuse by an attacker after stealing a password.

| Risk | Description | MITRE ATT&CK Reference |
|---|---|---|
| Root Password Brute Force via su | Repeated attempts to guess the root password through `su -` | T1110 - Brute Force |
| Privilege Escalation after Compromise | An attacker who has compromised a regular user account tries to use `su` to reach root if they know the password (or have exploited a vulnerability) | T1548 - Abuse Elevation Control Mechanism |
| Lateral Movement via su | Using `su` to move between different Service Accounts on the same machine to expand the scope of access | T1078 - Valid Accounts |
| Environment Manipulation (Non-Login Shell Abuse) | Deliberately abusing a Non-Login Shell to take advantage of old Environment Variables and run malicious code with higher privileges | T1574 - Hijack Execution Flow |

> [!WARNING]
> Every `su` attempt, **successful or failed**, is usually logged in `/var/log/auth.log` (on Debian/Ubuntu systems) or `/var/log/secure` (on RHEL/CentOS systems). A large number of consecutive failed `su` attempts on the root account is a classic indicator of a **Brute Force Attack**.

### Detection & Best Practices

- Monitor `/var/log/auth.log` or `/var/log/secure` for repeated failed `su` attempts
- Restrict the use of `su` by using a tool such as **`sudo` with specific privileges (Least Privilege)** instead of giving the root password to a large number of users
- Enable **`pam_wheel`** to restrict using `su` to switch to root to a specific group of trusted users only
- Review user-switching logs regularly to make sure every use of `su` has a documented, known reason

> [!TIP]
> If you notice in the Logs that a regular user successfully ran `su` to root at an unusual time (such as the middle of the night) or from an unfamiliar SSH session, it deserves immediate investigation - especially if that account has no prior history of using administrative privileges.

---

## Summary

- `su` lets you temporarily switch to another user (or root), or run a single command with their privileges
- **The key difference**: `su -` (Login Shell) loads the new user's full Environment and takes you to their Home Directory, while plain `su` (Non-Login Shell) leaves you in the same old Environment and location
- **The main recommendation**: Always use `su -` unless you have a clear reason to do otherwise
- From a SOC standpoint: `su` is a key monitoring point for **Privilege Escalation (T1548)** and **Brute Force (T1110)**, and reviewing `/var/log/auth.log` or `/var/log/secure` is essential for detecting any suspicious attempts
