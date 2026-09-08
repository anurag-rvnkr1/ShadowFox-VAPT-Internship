# 🔒 Vulnerability Assessment Report — ShadowFox Cyber Security Internship

This document summarizes the security vulnerabilities identified during the **TryHackMe Basic Pentesting** assessment completed as part of the **ShadowFox Cyber Security Internship**.

The assessment was conducted in an **authorized educational lab environment** using Kali Linux and standard penetration testing tools. Each finding includes its description, impact, risk rating, supporting evidence, and recommended remediation.

> **Assessment Type:** Vulnerability Assessment & Penetration Testing (VAPT)

---

# Executive Summary

During the penetration testing assessment, multiple security weaknesses were identified across the target Linux system. The vulnerabilities ranged from **information disclosure** to **critical privilege escalation** caused by insecure configurations and weak authentication practices.

## Assessment Summary

| Category                        | Count |
| ------------------------------- | ----: |
| Critical Vulnerabilities        | **1** |
| High Severity Vulnerabilities   | **4** |
| Medium Severity Vulnerabilities | **1** |
| Low Severity Vulnerabilities    | **0** |

---

# Severity Rating Scale

| Severity    | Description                                                             |
| ----------- | ----------------------------------------------------------------------- |
| 🔴 Critical | Immediate compromise of sensitive resources or privilege escalation.    |
| 🟠 High     | Significant security weakness that could lead to unauthorized access.   |
| 🟡 Medium   | Information disclosure or insecure configuration requiring remediation. |
| 🟢 Low      | Minor security issue with limited impact.                               |

---

# Finding 1 — Anonymous SMB Share Enabled

**Severity:** 🔴 High

**Category:** Authentication / SMB Misconfiguration

## Description

The SMB service allowed anonymous users to enumerate and access publicly shared resources without providing valid credentials.

During SMB enumeration, an anonymous share was accessible, allowing retrieval of internal files containing user-related information.

## Risk Impact

* Unauthorized users can access shared files.
* Internal usernames become exposed.
* Provides attackers with information useful for further attacks.

## Evidence

### Enumeration Tool

* Enum4Linux
* SMBClient

### Enumeration Activity

* SMB shares listed without authentication.
* Anonymous share successfully accessed.
* Internal file retrieved from the share.

## Security Impact

Information obtained from the anonymous share contributed directly to discovering valid usernames used during later authentication attempts.

## Recommendation

* Disable anonymous SMB access.
* Require authentication for all SMB shares.
* Limit SMB share permissions using the principle of least privilege.
* Regularly audit shared resources.

---

# Finding 2 — Exposed Development Directory

**Severity:** 🟡 Medium

**Category:** Information Disclosure

## Description

A publicly accessible development directory was discovered during web enumeration.

The directory contained developer notes and internal project files that should not have been exposed through the web server.

## Risk Impact

* Sensitive implementation details exposed.
* Internal technology stack revealed.
* User-related hints disclosed.
* Assists attackers during reconnaissance.

## Evidence

### Discovery Method

HTTP enumeration identified a hidden directory accessible over the web server.

### Files Observed

* `dev.txt`
* `j.txt`

These files contained developer notes and internal references useful during enumeration.

## Security Impact

Exposed development artifacts reduce the effort required for attackers during reconnaissance and vulnerability discovery.

## Recommendation

* Disable Apache directory indexing.
* Remove development files from production environments.
* Restrict access to development resources.
* Review web server configuration before deployment.

---

# Finding 3 — Weak SSH Password Policy

**Severity:** 🔴 High

**Category:** Authentication Weakness

## Description

A valid SSH password was successfully identified using a dictionary attack against one of the discovered user accounts.

The password matched an entry from a commonly used password dictionary.

## Risk Impact

* Unauthorized SSH access.
* Remote authenticated shell.
* Potential lateral movement.
* Increased privilege escalation opportunities.

## Evidence

### Tool Used

Hydra

### Attack Technique

Dictionary attack using the RockYou password list.

## Security Impact

Weak passwords remain one of the most common causes of unauthorized access. Dictionary attacks can compromise accounts if password complexity requirements are not enforced.

## Recommendation

* Enforce strong password complexity.
* Enable account lockout after repeated failed logins.
* Use multi-factor authentication where possible.
* Monitor authentication logs for brute-force attempts.

---

# Finding 4 — Linux User Enumeration Exposure

**Severity:** 🟠 High

**Category:** Information Disclosure

## Description

After obtaining authenticated access, system user information was easily enumerated through standard Linux configuration files.

Enumeration revealed additional user accounts present on the machine.

## Risk Impact

* Identification of privileged users.
* Enumeration of home directories.
* Supports privilege escalation planning.

## Evidence

### Enumeration Method

Inspection of Linux user configuration.

### Information Identified

* Multiple user accounts.
* Home directory locations.
* Login shell information.

## Security Impact

User enumeration assists attackers in identifying potential targets for lateral movement or privilege escalation.

## Recommendation

* Restrict unnecessary information exposure.
* Review local account permissions.
* Remove unused user accounts.
* Implement least privilege access policies.

---

# Finding 5 — Exposed SSH Private Key

**Severity:** 🔴 Critical

**Category:** Credential Exposure / Privilege Escalation

## Description

A private SSH key belonging to another user was accessible due to insecure file permissions and directory configuration.

The key could be copied for offline analysis.

## Risk Impact

* Authentication material exposed.
* User impersonation possible.
* Privilege escalation achieved.

## Evidence

### Sensitive Files Identified

* `id_rsa`
* `id_rsa.pub`
* `authorized_keys`

### Enumeration Activity

Inspection of another user's SSH configuration revealed accessible authentication files.

## Security Impact

Exposure of private keys is considered a critical security issue because it allows attackers to authenticate without knowing the account password once the key or passphrase is compromised.

## Recommendation

* Restrict `.ssh` directory permissions (`700`).
* Restrict private key permissions (`600`).
* Rotate exposed SSH keys immediately.
* Monitor unauthorized SSH key usage.

---

# Finding 6 — Weak SSH Key Passphrase

**Severity:** 🔴 High

**Category:** Cryptographic Weakness

## Description

The encrypted SSH private key was protected with a passphrase that was vulnerable to a dictionary attack.

The passphrase was recovered using password-cracking techniques.

## Risk Impact

* Encrypted key protection bypassed.
* Unauthorized authentication possible.
* Privilege escalation facilitated.

## Evidence

### Tools Used

* ssh2john.py
* John the Ripper

### Attack Method

Dictionary attack against converted SSH key hash.

## Security Impact

Encrypted SSH keys provide additional security only if protected with sufficiently strong passphrases.

## Recommendation

* Use long and unique SSH key passphrases.
* Avoid dictionary-based words.
* Rotate compromised keys.
* Store SSH keys securely.

---

# Finding 7 — Sensitive Backup Password File

**Severity:** 🔴 High

**Category:** Sensitive Information Exposure

## Description

A backup password file existed within another user's home directory.

After privilege escalation, the file became readable and contained sensitive authentication information.

## Risk Impact

* Credentials exposed in plaintext.
* Additional account compromise possible.
* Sensitive secrets stored insecurely.

## Evidence

### Sensitive File

`pass.bak`

### Observation

The file became accessible only after successful privilege escalation.

## Security Impact

Storing passwords in plaintext backup files creates unnecessary credential exposure risks.

## Recommendation

* Never store passwords in plaintext.
* Use encrypted secret storage solutions.
* Restrict access permissions.
* Remove obsolete backup credential files.

---

# Attack Path Summary

The assessment followed the following progression:

```text
Reconnaissance
      │
      ▼
Port & Service Enumeration
      │
      ▼
Web Enumeration
      │
      ▼
SMB Enumeration
      │
      ▼
Username Discovery
      │
      ▼
SSH Authentication
      │
      ▼
Linux Enumeration
      │
      ▼
SSH Key Discovery
      │
      ▼
SSH Key Passphrase Recovery
      │
      ▼
Privilege Escalation
      │
      ▼
Access to Restricted Resources
```

This demonstrates how multiple individually moderate vulnerabilities can combine into a successful privilege escalation chain.

---

# Risk Assessment Matrix

| Finding                        | Severity    | CIA Impact                                 |
| ------------------------------ | ----------- | ------------------------------------------ |
| Anonymous SMB Share            | 🔴 High     | Confidentiality                            |
| Exposed Development Directory  | 🟡 Medium   | Confidentiality                            |
| Weak SSH Password              | 🔴 High     | Confidentiality / Integrity                |
| Linux User Enumeration         | 🟠 High     | Confidentiality                            |
| Exposed SSH Private Key        | 🔴 Critical | Confidentiality / Integrity / Availability |
| Weak SSH Key Passphrase        | 🔴 High     | Confidentiality                            |
| Sensitive Backup Password File | 🔴 High     | Confidentiality                            |

---

# Mitigation Summary

| Vulnerability              | Recommended Mitigation                                            |
| -------------------------- | ----------------------------------------------------------------- |
| Anonymous SMB Access       | Disable guest access and enforce authentication.                  |
| Directory Listing          | Disable Apache directory indexing and remove development files.   |
| Weak Password Policy       | Enforce password complexity and account lockout policies.         |
| User Enumeration           | Minimize unnecessary account exposure and remove unused users.    |
| Exposed SSH Private Keys   | Restrict permissions, rotate keys, and secure `.ssh` directories. |
| Weak SSH Passphrase        | Use strong, randomly generated passphrases for SSH keys.          |
| Plaintext Backup Passwords | Store secrets securely using encrypted credential management.     |

---

# Lessons Learned

This assessment demonstrates several important VAPT concepts:

* Misconfigurations can expose sensitive internal information.
* Enumeration is one of the most important phases of penetration testing.
* Weak authentication significantly increases attack surface.
* Linux file permissions play a critical role in system security.
* SSH key management is essential for secure remote access.
* Multiple small vulnerabilities can combine into a complete compromise path.

---

# Responsible Disclosure Statement

This assessment was completed **only within the TryHackMe Basic Pentesting laboratory** during the **ShadowFox Cyber Security Internship**.

All findings were identified in a controlled educational environment designed for cybersecurity training and ethical hacking practice. This repository is intended solely for educational, documentation, and portfolio purposes.
