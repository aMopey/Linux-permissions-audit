# Linux Permission Using Symbolic & Numeric Methods

**Author:** Rukayat Mopelola Lawal  
**Platform:** Kali Linux (Oracle VirtualBox)

---

## Overview

A hands-on Linux administration assignment covering two core areas: file permission management using both symbolic and numeric `chmod` methods, and user/group administration with access-controlled shared directories. All tasks were completed on a Kali Linux virtual machine.

---

## Section 1: File Permission Assignment

### Scenario

Acting as Linux administrator for a small team, the task was to inspect and modify permissions on a shared file (`report.txt`) using both permission methods.

### Tasks Completed

**Step 1 — Create and verify the file**

```bash
touch report.txt
ls
```

**Step 2 — Check initial permissions**

```bash
ls -l
# -rw------- 1 kali kali 0 Apr 20 16:32 report.txt
```

**Step 3 — Set permissions using the symbolic method**

Target: owner = read/write, group = read only, others = no permission

```bash
chmod g+w report.txt   # add write temporarily
chmod o-r report.txt   # remove read from others
```

Result: `-rw-r-----`

**Step 4 — Set permissions using the numeric method**

Target: owner = read/write/execute, group = read/execute, others = no permission

```bash
chmod 750 report.txt
```

Result: `-rwxr-x---`

### Permission Results Summary

| Task | Method | Permission String | Numeric Equivalent |
|------|--------|------------------|--------------------|
| Owner rw, Group r, Others none | Symbolic | `-rw-r-----` | 640 |
| Owner rwx, Group rx, Others none | Numeric | `-rwxr-x---` | 750 |

### Reflection: Symbolic vs. Numeric Methods

**Symbolic Method** — uses letters to describe who, what action, and which permission:

| Component | Options |
|-----------|---------|
| Who | `u` (user/owner), `g` (group), `o` (others), `a` (all) |
| Action | `+` (add), `-` (remove), `=` (set exactly) |
| Permission | `r` (read), `w` (write), `x` (execute) |

Example: `chmod g+r,o-rwx report.txt`

**Numeric Method** — uses octal values to set all permissions at once:

| Value | Permission |
|-------|------------|
| 4 | Read |
| 2 | Write |
| 1 | Execute |

Each digit represents owner, group, and others respectively. Example: `chmod 750` = `rwxr-x---`

---

## Section 2: User and Group Administration

### Scenario

Preparing a Linux system for two departments — **HR** and **IT** — by creating groups, users, assigning memberships, and enforcing directory-level access controls.

### Tasks Completed

**Step 1 — Create groups**

```bash
sudo groupadd HR
sudo groupadd IT
```

**Step 2 — Create users and assign passwords**

```bash
sudo adduser Amina   # assigned to HR
sudo adduser Kola    # assigned to IT
```

**Step 3 — Add users to their groups**

```bash
sudo usermod -aG HR Amina
sudo usermod -aG IT Kola
```

**Step 4 — Verify group membership**

```bash
id Amina
# uid=1003(Amina) gid=1003(Amina) groups=1003(Amina),100(users),1001(HR)

id Kola
# uid=1004(Kola) gid=1004(Kola) groups=1004(Kola),100(users),1002(IT)
```

**Step 5 — Create shared directories**

```bash
mkdir -p /shared/hr_docs
mkdir -p /shared/it_docs
```

**Step 6 — Set group ownership**

```bash
sudo chgrp HR /shared/hr_docs
sudo chgrp IT /shared/it_docs
```

**Step 7 — Restrict directory access (owner + group only)**

```bash
sudo chmod 770 /shared/hr_docs
sudo chmod 770 /shared/it_docs
```

Result:

```
drwxrwx--- 2 root HR 4096 Apr 20 14:45 hr_docs
drwxrwx--- 2 root IT 4096 Apr 20 14:45 it_docs
```

**Step 8 — Modify user group membership**

Add Amina to IT (additional group):

```bash
sudo usermod -aG IT Amina
# groups Amina → Amina : Amina users HR IT
```

Remove Amina from IT:

```bash
sudo gpasswd -d Amina IT
# Removing user Amina from group IT
```

**Step 9 — Verify access control**

```bash
sudo -u Amina ls /shared/hr_docs   # ✓ Access granted
sudo -u Amina ls /shared/it_docs   # ✗ Permission denied

sudo -u Kola ls /shared/it_docs    # ✓ Access granted
sudo -u Kola ls /shared/hr_docs    # ✗ Permission denied
```

### Access Control Summary

| Directory | Group Owner | Permissions | HR User (Amina) | IT User (Kola) |
|-----------|-------------|-------------|-----------------|----------------|
| `/shared/hr_docs` | HR | `rwxrwx---` | ✓ Access Granted | ✗ Access Denied |
| `/shared/it_docs` | IT | `rwxrwx---` | ✗ Access Denied | ✓ Access Granted |

### Key Concepts

Linux evaluates permissions in order: **User → Group → Others**

- Every file/directory has a group owner
- When a user is added to a group, they inherit that group's permission set instead of falling into the restricted "Others" category
- Directories require **execute (`x`)** permission to enter — without it, `cd` and `ls` are blocked even if read permission exists

---

## Commands Reference

| Command | Purpose |
|---------|---------|
| `chmod [symbolic] file` | Change permissions using symbolic notation |
| `chmod [octal] file` | Change permissions using numeric notation |
| `ls -l` | List files with permission strings |
| `groupadd <name>` | Create a new group |
| `adduser <name>` | Create a new user interactively |
| `usermod -aG <group> <user>` | Add user to a group |
| `gpasswd -d <user> <group>` | Remove user from a group |
| `chgrp <group> <path>` | Change group ownership of a file/directory |
| `id <user>` | Show user ID and group memberships |
| `groups <user>` | List groups a user belongs to |
