# 🛡️ ShadowFox Cyber Security Internship — VAPT Project (TryHackMe Basic Pentesting)

<div align="center">

### Vulnerability Assessment & Penetration Testing (VAPT) | Ethical Hacking | Kali Linux | TryHackMe

*Hands-on penetration testing project completed during the ShadowFox Cyber Security Internship.*

![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)
![Domain](https://img.shields.io/badge/Domain-Cyber%20Security-red?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-TryHackMe-212C42?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Kali%20Linux-557C94?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-VAPT-orange?style=for-the-badge)

</div>

---

## 📌 Project Overview

This repository documents my **Vulnerability Assessment and Penetration Testing (VAPT)** project completed as part of the **ShadowFox Cyber Security Internship**.

The assessment was performed in a **legally authorized TryHackMe lab environment** using **Kali Linux** against the **Basic Pentesting** virtual machine. The project follows a real-world penetration testing methodology, beginning with reconnaissance and ending with privilege escalation and security reporting.

Rather than exploiting a production system, this project demonstrates practical offensive security skills in a controlled environment designed for cybersecurity training.

### Key Highlights

* Network reconnaissance and service enumeration.
* Web application enumeration.
* SMB security assessment.
* Credential discovery through dictionary attacks.
* Linux enumeration and privilege escalation.
* SSH key analysis and password cracking.
* Security findings with mitigation recommendations.

---

# 🎯 Internship Information

| **Field**                   | **Details**                                    |
| --------------------------- | ---------------------------------------------- |
| **Internship Organization** | ShadowFox                                      |
| **Domain**                  | Cyber Security                                 |
| **Project**                 | Vulnerability Assessment & Penetration Testing |
| **Lab Platform**            | TryHackMe                                      |
| **Room**                    | Basic Pentesting                               |
| **Operating System**        | Kali Linux                                     |
| **Project Status**          | ✅ Successfully Completed                       |

---

# 🧠 Skills Demonstrated

<table>
<tr>
<td width="50%">

### Offensive Security

* Penetration Testing Methodology
* Network Reconnaissance
* Service Enumeration
* SMB Enumeration
* Credential Attacks
* Privilege Escalation
* SSH Authentication

</td>

<td width="50%">

### Security Tools

* Kali Linux
* Nmap
* Enum4Linux
* SMBClient
* Hydra
* John the Ripper
* SSH
* OpenVPN

</td>
</tr>
</table>

---

# 🏗️ Lab Architecture

```text
                +---------------------------+
                |      TryHackMe Network    |
                +-------------+-------------+
                              |
                         OpenVPN Tunnel
                              |
                     tun0 Interface (VPN)
                              |
                  +-----------+-----------+
                  |                       |
          Kali Linux VM           Target Linux Machine
        (Attacking Machine)      (Basic Pentesting Lab)
```

The assessment was performed entirely inside the TryHackMe VPN environment using Kali Linux as the attacking machine.

---

# 🔄 Penetration Testing Methodology

This project follows a structured VAPT lifecycle similar to professional penetration testing engagements.

| Phase                   | Objective                                                         |
| ----------------------- | ----------------------------------------------------------------- |
| Reconnaissance          | Identify reachable target and exposed attack surface.             |
| Enumeration             | Gather detailed information about services, users, and resources. |
| Vulnerability Discovery | Identify insecure configurations and exposed assets.              |
| Credential Access       | Obtain valid credentials using authorized password attacks.       |
| Initial Access          | Authenticate into the target machine through SSH.                 |
| Privilege Escalation    | Enumerate Linux permissions and escalate privileges.              |
| Reporting               | Document findings, risks, and recommendations.                    |

---

# 🛠️ Tools & Technologies Used

| Tool                | Purpose                                                      |
| ------------------- | ------------------------------------------------------------ |
| **Kali Linux**      | Penetration testing operating system.                        |
| **OpenVPN**         | Secure connection to TryHackMe lab network.                  |
| **Nmap**            | Network discovery, service detection, and OS fingerprinting. |
| **Enum4Linux**      | SMB enumeration and user discovery.                          |
| **SMBClient**       | Access anonymous SMB shares.                                 |
| **Hydra**           | SSH dictionary attack against discovered users.              |
| **SSH**             | Remote authenticated shell access.                           |
| **ssh2john.py**     | Convert encrypted SSH private key into John-compatible hash. |
| **John the Ripper** | Crack SSH key passphrase using dictionary attack.            |

---

# 🚀 Assessment Workflow

## 1️⃣ Environment Setup

**Objective:** Connect Kali Linux to the TryHackMe VPN network.

### Activities

* Connected using OpenVPN configuration.
* Verified VPN tunnel (`tun0` interface).
* Confirmed connectivity with the target machine.

### Commands Used

```bash
sudo openvpn ~/Downloads/tryhackme.ovpn
ip a
ping <TARGET_IP>
```

📷 Screenshot: `screenshots/01-openvpn.png`

---

## 2️⃣ Network Reconnaissance

**Objective:** Identify open ports, running services, and operating system information.

### Tool Used

* Nmap

### Command

```bash
nmap -sV -A <TARGET_IP>
```

### Services Identified

| Port | Service | Purpose               |
| ---- | ------- | --------------------- |
| 22   | SSH     | Remote shell access.  |
| 80   | HTTP    | Apache web server.    |
| 139  | NetBIOS | SMB communication.    |
| 445  | SMB     | File sharing service. |

### Outcome

The scan identified multiple exposed services that became potential attack vectors during later stages of the assessment.

📷 Screenshot: `screenshots/02-nmap-scan.png`

---

## 3️⃣ Web Enumeration

**Objective:** Discover hidden web resources and sensitive files.

### Enumeration Method

* HTTP Enumeration
* Apache Directory Listing

### Command

```bash
sudo nmap -p80 --script http-enum <TARGET_IP>
```

### Findings

A hidden directory named:

```text
/development
```

Inside this directory:

* `dev.txt`
* `j.txt`

These files contained developer notes and user-related hints useful during enumeration.

### Security Observation

Developer files exposed internal information that should never be publicly accessible.

📷 Screenshot: `screenshots/03-development-directory.png`

---

## 4️⃣ SMB Enumeration

**Objective:** Assess SMB configuration and discover accessible resources.

### Tools Used

* Enum4Linux
* SMBClient

### Commands

```bash
enum4linux -a <TARGET_IP>

smbclient //<TARGET_IP>/Anonymous
```

### Findings

* Anonymous SMB access enabled.
* Public share accessible without authentication.
* Downloaded `staff.txt`.

### Information Extracted

Valid usernames discovered:

* `jan`
* `kay`

### Security Impact

Anonymous SMB shares exposed sensitive internal user information.

📷 Screenshots

* `screenshots/04-enum4linux.png`
* `screenshots/05-smbclient.png`

---

## 5️⃣ Credential Discovery (SSH Password Attack)

**Objective:** Identify valid SSH credentials using dictionary attack.

### Tool Used

Hydra

### Command

```bash
hydra -l jan -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP> -I
```

### Result

Valid SSH credentials obtained for the user `jan`.

### Security Observation

The target used a weak password vulnerable to dictionary attacks.

📷 Screenshot: `screenshots/06-hydra-success.png`

---

## 6️⃣ Initial Access via SSH

**Objective:** Obtain authenticated shell access.

### Command

```bash
ssh jan@<TARGET_IP>
```

### Activities Performed

* Logged into the Linux machine.
* Verified shell access.
* Enumerated local users.
* Inspected `/etc/passwd`.

### Outcome

Discovered another user account named `kay`.

📷 Screenshot: `screenshots/07-ssh-login.png`

---

## 7️⃣ Linux Enumeration

**Objective:** Identify privilege escalation opportunities.

### Enumeration Activities

* Checked user home directories.
* Listed file permissions.
* Examined hidden files.
* Investigated SSH configuration.

### Important Findings

Located inside `/home/kay`:

* `pass.bak`
* `.ssh`
* `authorized_keys`
* `id_rsa`
* `id_rsa.pub`

### Security Observation

Sensitive authentication material was exposed due to improper permissions.

📷 Screenshot: `screenshots/08-linux-enumeration.png`

---

## 8️⃣ SSH Private Key Analysis

**Objective:** Analyze and recover credentials from encrypted SSH key.

### Steps Performed

1. Copied encrypted SSH private key.
2. Protected local copy using file permissions.
3. Converted SSH key into hash format.

### Commands

```bash
chmod 400 bp

locate ssh2john.py

python3 ssh2john.py bp > hash.txt
```

### Outcome

Generated a password hash compatible with John the Ripper.

📷 Screenshot: `screenshots/09-ssh2john.png`

---

## 9️⃣ Password Cracking

**Objective:** Recover the SSH private key passphrase.

### Tool Used

John the Ripper

### Command

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

### Result

Recovered the encrypted SSH key passphrase using a dictionary attack.

### Security Observation

Weak SSH key passphrases significantly reduce the effectiveness of key-based authentication.

📷 Screenshot: `screenshots/10-john-crack.png`

---

## 🔟 Privilege Escalation

**Objective:** Authenticate as the second user using the recovered private key.

### Command

```bash
ssh -i bp kay@<TARGET_IP>
```

### Activities

* Logged in as `kay`.
* Accessed previously restricted files.
* Retrieved sensitive backup credentials.

### Outcome

Successfully escalated privileges and completed the assessment objectives.

📷 Screenshot: `screenshots/11-ssh-kay.png`

---

# 📊 Vulnerability Assessment Summary

| Vulnerability                  | Severity    | Risk                                      |
| ------------------------------ | ----------- | ----------------------------------------- |
| Anonymous SMB Share            | 🔴 High     | Unauthorized access to shared resources.  |
| Exposed Development Directory  | 🟠 Medium   | Information disclosure.                   |
| Weak SSH Password              | 🔴 High     | Dictionary attack successful.             |
| Exposed SSH Private Key        | 🔴 Critical | Authentication material exposed.          |
| Weak SSH Key Passphrase        | 🔴 High     | Private key compromised through cracking. |
| Sensitive Backup Password File | 🔴 High     | Credential exposure.                      |

---

# 🛡️ Mitigation Recommendations

| Security Finding          | Recommended Mitigation                                         |
| ------------------------- | -------------------------------------------------------------- |
| Anonymous SMB Share       | Disable guest access and require authentication.               |
| Directory Listing Enabled | Disable Apache directory indexing and remove sensitive files.  |
| Weak Password Policy      | Enforce strong password complexity and account lockout.        |
| Exposed SSH Keys          | Restrict permissions (`chmod 600`) and protect private keys.   |
| Weak SSH Passphrase       | Use strong, unique passphrases for encrypted SSH keys.         |
| Sensitive Backup Files    | Store credentials securely using encrypted secrets management. |

---

# 📚 Learning Outcomes

Through this internship project I gained practical experience in:

### Technical Skills

* Network scanning and reconnaissance.
* Service and version enumeration.
* SMB security assessment.
* Linux user enumeration.
* SSH authentication workflow.
* Password cracking fundamentals.
* Privilege escalation methodology.

### Practical Security Knowledge

* Identifying insecure configurations.
* Analyzing Linux permissions.
* Understanding SSH key authentication.
* Conducting structured VAPT assessments.
* Writing professional vulnerability reports.

---

# 📂 Repository Structure

```text
ShadowFox-VAPT-Internship/
│
├── README.md
├── internship-certificate.pdf
├── report/
│   └── ShadowFox_VAPT_Report.pdf
├── screenshots/
│   ├── 01-openvpn.png
│   ├── 02-nmap-scan.png
│   ├── 03-development-directory.png
│   ├── 04-enum4linux.png
│   ├── 05-smbclient.png
│   ├── 06-hydra-success.png
│   ├── 07-ssh-login.png
│   ├── 08-linux-enumeration.png
│   ├── 09-ssh2john.png
│   ├── 10-john-crack.png
│   └── 11-ssh-kay.png
├── commands/
│   └── commands.md
└── findings/
    └── vulnerabilities.md
```

---

# 📖 Documentation

This repository includes:

* Practical VAPT workflow documentation.
* Commands used during the assessment.
* Screenshots of each phase.
* Vulnerability analysis.
* Mitigation recommendations.
* Internship completion certificate.

---

# ⚠️ Ethical Use Disclaimer

This project was completed **only inside the TryHackMe training environment** as part of the **ShadowFox Cyber Security Internship**.

All penetration testing activities were performed against an intentionally vulnerable machine within an authorized lab environment for educational purposes. The techniques documented in this repository should only be used on systems for which explicit authorization has been granted.

---

# 👨‍💻 About Me

**Anurag**

Aspiring Cyber Security Engineer focused on **Vulnerability Assessment & Penetration Testing (VAPT)**, Linux Security, Web Application Security, and Ethical Hacking.


---

<div align="center">

### ⭐ If you found this project interesting, feel free to star the repository.

*Cyber Security Portfolio Project • ShadowFox Internship • TryHackMe Basic Pentesting*

</div>

