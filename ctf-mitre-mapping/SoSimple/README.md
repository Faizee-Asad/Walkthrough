# SoSimple CTF - MITRE ATT&CK Mapping & Walkthrough

**Category:** Boot2Root  
**Difficulty:** Easy  
**Tools Used:** Nmap, Feroxbuster, WPScan, Python HTTP server, SSH  

---

## 🔍 Summary

This challenge involves exploiting a vulnerable WordPress installation using a known Social Warfare plugin vulnerability, followed by privilege escalation through misconfigured sudo permissions and a root-owned script.

---

## 🧠 MITRE ATT&CK Mapping

| Phase | Technique | ID | Description |
|-------|-----------|----|-------------|
| Reconnaissance | Active Scanning | T1595.002 | Used `nmap` and `feroxbuster` to discover open ports and WordPress directory |
| Credential Access | Brute Force | T1110.001 | Used `wpscan` to brute-force WordPress login for user `max` |
| Initial Access | Exploit Public-Facing Application | T1190 | Exploited Social Warfare plugin to run arbitrary PHP code |
| Execution | Command and Scripting Interpreter (PHP) | T1059.004 | Executed `system("bash -c ...")` via a crafted payload |
| Credential Access | Unsecured Credentials | T1552.001 | Retrieved `id_rsa` SSH key from /home/max/.ssh |
| Persistence | Valid Accounts | T1078 | Logged in via SSH using retrieved private key |
| Privilege Escalation | Abuse Elevation Control Mechanism | T1548.003 | Abused `sudo -u steven` and path traversal |
| Privilege Escalation | Abuse Elevation Control Mechanism | T1548.002 | Root-owned script executed via `sudo` |
| Discovery | Permission Groups Discovery | T1069.001 | Enumerated sudo rights using `sudo -l` |
| Collection | Data from Local System | T1005 | Retrieved `user.txt` and `flag.txt` |

---

## 🔧 Steps

### 1. Enumeration

```bash
ping 192.168.210.78
nmap -sC -sV -Pn 192.168.210.78
