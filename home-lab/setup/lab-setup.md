# Room/Machine Name

**Platform:** TryHackMe  
**Difficulty:** Easy  
**Date:** 2026-04-21

## Summary
Setup and installed VirtualBox for VMs. I am using Kali linux and Metasploitable, a version of Ubuntu setup to be a punchingbag for cybersec labs

## What I Learned
- How to setup and install VMs in Virtualbox
- How to setup local networks within Virtualbox
- How to obtain and verify connectivity between the two VMs
## Walkthrough

### Step 1 - Reconnaissance
What I did and why:
```bash
nmap -sV 192.168.56.101
```
This allowed me to find the IP of my connected machines within my VM network

### Step 2 - Exploitation
...

## Key Commands Reference
| Command | What it does |
|---|---|
| `nmap -sV <ip>` | Scan for open ports and service versions |
