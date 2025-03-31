# Linux Privilege Escalation Enumeration

## Basic Enumeration

- **`cat /etc/os-release`**  
  To identify the OS version and check if it is vulnerable.

- **`echo $PATH`**  
  To view the current PATH variable. You can add a new path to execute commands using:  
  `export PATH=$PATH:New_Path`

- **`env`**  
  To list the environment variables of the Linux machine.

---

## Kernel Exploit

- **`cat /proc/version`**  
  To determine the kernel version.

- **`uname -a`**  
  To get detailed information about the kernel version.

- **`lscpu`**  
  To gather more information about the CPU.

- **`cat /etc/shells`**  
  To list available shells on the system.

---

## Permissions

### SUID and SGID Bits
- **Setuid (SUID)**:  
  When set, files will execute with the privileges of their owner.

- **Setgid (SGID)**:  
  When set, files will execute with the privileges of their group. If set on a directory, files created within that directory will inherit the group of the directory itself.

### File Permissions
- The format `-rwxrwxrwx`:  
  - The first `rwx` is for the owner.  
  - The second `rwx` is for the group.  
  - The third `rwx` is for others.

- SUID/SGID permissions are represented by an `'s'` in the execute position.  

### SETUID and SETGID Permissions
Binaries with these permissions allow a user to run a command as root without granting root-level access directly. For example:
```
ahmed💀ahmed:~$ ls -lah /bin/passwd
-rwsr-xr-x 1 root root 116K Dec 6 07:51 /bin/passwd
```
The letter `s` in place of `x` indicates that anyone can read, write, and execute with all permissions as the owner.

---

## Finding Files and Directories

### Find All Files with SETUID Permissions
Use:
```
find / -perm -4000 -o -perm -2000 -type f 2>/dev/null
```

#### Explanation:
- **`find /`:** Starts searching from the root directory (`/`) and includes all subdirectories.
- **`-perm -4000`:** Matches files with the SUID (Set User ID) bit set, allowing execution with the file owner's permissions.
- **`-o`:** Logical OR operator.
- **`-perm -2000`:** Matches files with SGID (Set Group ID) bit set, allowing execution with group permissions.
- **`-type f`:** Restricts search to regular files only.
- **`2>/dev/null`:** Silences error messages (e.g., "Permission denied").

---

### Find All Writable Directories
Use:
```
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```

#### Explanation:
- **`find /`:** Starts searching from the root directory (`/`) and includes all subdirectories.
- **`-path /proc`:** Specifies `/proc`, a virtual filesystem, to exclude from the search.
- **`-prune`:** Prevents descending into `/proc`.
- **`-o`:** Logical OR operator.
- **`-type d`:** Restricts search to directories only.
- **`-perm -o+w`:** Matches directories that are world-writable (writable by any user).
- **`2>/dev/null`:** Silences error messages (e.g., "Permission denied").

---

## Managing SUID Permissions

### Set SUID Bit
Use:
```
chmod u+s filename
```

### Remove SUID Bit
Use:
```
chmod u-s filename
```

---

## Groups and Users

To find groups and users in those groups, use:
```
cat /etc/group
```
This lists all groups on the system along with their associated users.
```
To find the users in the group, use:
```
ahmed💀ahmed:~$  getent group sudo(group name)
sudo:x:27:ahmed
```
