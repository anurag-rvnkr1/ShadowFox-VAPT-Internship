# 🛡️ ShadowFox Cyber Security Internship – Complete VAPT Documentation

## Project: TryHackMe Basic Pentesting (Vulnerability Assessment & Penetration Testing)

**Intern:** Anurag

**Organization:** ShadowFox

**Domain:** Cyber Security (VAPT)

**Target Machine IP:** `10.10.43.214`

**Platform:** TryHackMe

**Operating System Used:** Kali Linux

---

# 1. Project Overview

## 1.1 Introduction

As part of my Cyber Security Internship at **ShadowFox**, I completed a practical Vulnerability Assessment and Penetration Testing (VAPT) project using the **TryHackMe Basic Pentesting** room.

The project simulated a real-world penetration testing engagement against an intentionally vulnerable Linux machine hosted inside the TryHackMe environment. The objective was to identify vulnerabilities, gain authorized access through discovered weaknesses, enumerate the target system, and successfully perform privilege escalation using ethical hacking techniques.

Unlike theoretical cybersecurity exercises, this assessment followed a structured penetration testing methodology similar to professional VAPT engagements.

---

## 1.2 Project Objectives

The primary objectives of this assessment were:

* Establish secure connectivity with the target lab.
* Perform network reconnaissance.
* Enumerate exposed services.
* Identify hidden web resources.
* Enumerate SMB shares and users.
* Discover valid credentials.
* Gain authenticated SSH access.
* Perform Linux privilege escalation.
* Document vulnerabilities and remediation recommendations.

---

## 1.3 Assessment Scope

| Parameter         | Value                            |
| ----------------- | -------------------------------- |
| Target Machine    | TryHackMe Basic Pentesting       |
| Target IP Address | `10.10.43.214`                   |
| Assessment Type   | Internal Penetration Testing Lab |
| Attacking Machine | Kali Linux                       |
| Network Access    | OpenVPN Tunnel                   |
| Authorization     | TryHackMe Controlled Environment |

---

## 1.4 VAPT Methodology Used

This assessment followed a structured Vulnerability Assessment and Penetration Testing workflow.

```text
Reconnaissance
      │
      ▼
Network Enumeration
      │
      ▼
Web Enumeration
      │
      ▼
SMB Enumeration
      │
      ▼
Credential Discovery
      │
      ▼
SSH Authentication
      │
      ▼
Linux Enumeration
      │
      ▼
Privilege Escalation
      │
      ▼
Security Findings & Reporting
```

Every stage generated information that influenced the next step of the assessment.

---

# 2. Lab Environment Setup

## 2.1 Objective

Before beginning the penetration testing assessment, it was necessary to connect the attacking machine (Kali Linux) to the isolated TryHackMe laboratory network.

The target machine was **not publicly accessible** over the internet. Communication was possible only after establishing a VPN tunnel using OpenVPN.

---

## 2.2 Attack Environment

| Component         | Description                  |
| ----------------- | ---------------------------- |
| Attacking Machine | Kali Linux Virtual Machine   |
| Target Machine    | Linux VM hosted on TryHackMe |
| Connection Method | OpenVPN                      |
| VPN Interface     | `tun0`                       |

---

## 2.3 Connecting to the VPN

The VPN configuration file downloaded from the TryHackMe dashboard was used to establish connectivity.

### Command Used

```bash
sudo openvpn ~/Downloads/tryhackme.ovpn
```

### Command Explanation

| Component | Description                                                            |
| --------- | ---------------------------------------------------------------------- |
| `sudo`    | Executes OpenVPN with administrative privileges.                       |
| `openvpn` | Starts the VPN client.                                                 |
| `.ovpn`   | Configuration file containing VPN certificates and connection details. |

### Expected Result

After executing the command, the VPN client initialized the encrypted tunnel.

Successful connection was confirmed by the message:

> `Initialization Sequence Completed`

---

### 📷 Screenshot Placeholder

**Figure 2.1 — Successful OpenVPN Connection**

> *Insert screenshot showing the OpenVPN terminal with "Initialization Sequence Completed".*

---

## 2.4 Verifying the VPN Tunnel

After connecting to the VPN, I verified that Kali Linux created a new virtual network interface.

### Command Used

```bash
ip a
```

### Purpose

This command lists all available network interfaces on the attacking machine.

### Observation

A new interface named **tun0** appeared with an assigned VPN IP address.

### Analysis

The presence of the `tun0` interface confirmed that Kali Linux was connected to the TryHackMe network.

Without this interface, the target machine would not be reachable.

---

### 📷 Screenshot Placeholder

**Figure 2.2 — tun0 Interface Verification**

> *Insert screenshot showing output of `ip a` highlighting the `tun0` interface.*

---

## 2.5 Verifying Target Reachability

Before performing reconnaissance, I confirmed that the target machine responded through the VPN.

### Command Used

```bash
ping 10.10.43.214
```

### Purpose

Checks whether the target machine is reachable through ICMP packets.

### Observation

The target responded successfully with ICMP replies.

### Analysis

Successful responses confirmed:

* VPN connectivity.
* Target machine was online.
* Network communication was functioning correctly.

This completed the environment setup phase.

---

### 📷 Screenshot Placeholder

**Figure 2.3 — Successful Ping to Target Machine**

> *Insert screenshot showing ICMP replies from `10.10.43.214`.*

---

# 3. Phase 1 – Initial Reconnaissance

## 3.1 Objective

Reconnaissance is the first stage of penetration testing.

Its objective is to identify the attack surface by discovering:

* Open ports.
* Running services.
* Operating system information.
* Potential entry points.

Instead of immediately attempting exploitation, reconnaissance focuses on gathering information.

---

## 3.2 Why Nmap Was Used

Nmap (Network Mapper) is one of the most widely used penetration testing tools for network discovery.

It provides:

* Host discovery.
* Port scanning.
* Service detection.
* Version detection.
* OS fingerprinting.
* NSE script execution.

This makes it ideal for the initial enumeration phase.

---

## 3.3 Aggressive Service Enumeration Scan

### Command Used

```bash
nmap -sV -A 10.10.43.214
```

---

## 3.4 Command Breakdown

| Option         | Purpose                                                                                        |
| -------------- | ---------------------------------------------------------------------------------------------- |
| `-sV`          | Detect service versions running on open ports.                                                 |
| `-A`           | Enable aggressive scan including OS detection, traceroute, NSE scripts, and version detection. |
| `10.10.43.214` | Target machine IP.                                                                             |

---

## 3.5 Why This Scan Was Chosen

Instead of performing multiple scans individually, the aggressive scan collects a large amount of information in one execution.

This helps identify:

* Service versions.
* Operating system fingerprint.
* NetBIOS information.
* HTTP details.
* SMB security configuration.

---

### 📷 Screenshot Placeholder

**Figure 3.1 — Nmap Aggressive Scan Output**

> *Insert screenshot of the complete Nmap output.*

---

# 4. Enumeration Results

The Nmap scan successfully identified **four exposed TCP services**.

| Port | State | Service | Version                   |
| ---- | ----- | ------- | ------------------------- |
| 22   | Open  | SSH     | OpenSSH 8.2p1             |
| 80   | Open  | HTTP    | Apache HTTP Server 2.4.41 |
| 139  | Open  | NetBIOS | Samba                     |
| 445  | Open  | SMB     | Samba smbd v4             |

---

## 4.1 SSH Service Analysis (Port 22)

### Observation

SSH was running on port **22** using OpenSSH.

### Analysis

SSH provides encrypted remote shell access.

This immediately became a potential target because:

* SSH accepts username/password authentication.
* Credentials discovered later could be used for remote login.
* SSH often becomes the entry point after enumeration.

### Security Significance

At this stage, there were **no credentials**, so exploitation was not possible.

The SSH service was noted for future authentication attempts.

---

## 4.2 HTTP Service Analysis (Port 80)

### Observation

Apache HTTP Server was running.

### Analysis

Web servers frequently expose:

* Login portals.
* Hidden directories.
* Configuration files.
* Backup files.
* Development resources.

Therefore, web enumeration became the next logical step.

### Decision

Move to HTTP enumeration after completing network reconnaissance.

---

## 4.3 SMB Service Analysis (Ports 139 & 445)

### Observation

The target exposed SMB services through Samba.

### Analysis

SMB is commonly used for:

* File sharing.
* Printer sharing.
* User authentication.

Misconfigured SMB services often expose:

* Anonymous shares.
* Usernames.
* Shared documents.
* Configuration files.

This immediately indicated another potential attack vector.

---

## 4.4 Operating System Identification

Nmap fingerprinting indicated the target was running Linux.

### Additional Information Collected

* Linux Kernel Information
* Apache Version
* Samba Version
* NetBIOS Name (`BASIC2`)

### Analysis

Knowing the operating system helps determine:

* Available privilege escalation vectors.
* Linux-specific enumeration techniques.
* Relevant authentication mechanisms.

---

### 📷 Screenshot Placeholder

**Figure 3.2 — Open Services Identified by Nmap**

> *Insert screenshot highlighting ports 22, 80, 139, and 445.*

---

# 5. Analysis of Reconnaissance Results

Reconnaissance produced several important findings.

## Findings Summary

| Finding                   | Security Importance                               |
| ------------------------- | ------------------------------------------------- |
| SSH exposed               | Potential authentication target.                  |
| Apache web server exposed | Web enumeration required.                         |
| SMB exposed               | User/share enumeration possible.                  |
| Linux OS detected         | Linux privilege escalation techniques applicable. |

---

## Attack Surface Analysis

At this point, three independent attack surfaces existed:

### Attack Surface 1 — HTTP

Potential for:

* Hidden directories.
* Information disclosure.
* Web application vulnerabilities.

### Attack Surface 2 — SMB

Potential for:

* Anonymous shares.
* Username discovery.
* Sensitive file exposure.

### Attack Surface 3 — SSH

Potential for:

* Password authentication.
* Key-based authentication.
* Brute-force attack after user discovery.

---

## Why HTTP Enumeration Was Chosen Next

Although SMB looked promising, the Apache web server was selected for the next phase because:

1. Hidden directories often reveal sensitive information quickly.
2. Developer notes sometimes expose usernames or credentials.
3. Information disclosure can reduce the effort required during credential attacks.

This decision followed the penetration testing principle of gathering maximum intelligence before attempting authentication attacks.

---

# 🌐 Phase 2 – Web Enumeration & Information Gathering

> **Target:** `10.10.43.214`
>
> **Objective:** Enumerate the HTTP service running on the target machine, discover hidden directories, analyze publicly accessible files, and identify information disclosure vulnerabilities that could assist further penetration testing.

---

# 7. Web Enumeration

## 7.1 Objective

After completing the network reconnaissance phase, the next step was to enumerate the web server running on **Port 80**.

The Nmap scan confirmed that the target was running an **Apache HTTP Server**. Web servers frequently expose hidden directories, backup files, developer notes, configuration files, or sensitive resources that can provide valuable information during a penetration testing engagement.

Instead of immediately attempting exploitation, the goal during this phase was to gather as much intelligence as possible from the web application.

### Objectives of this Phase

* Inspect the web application.
* Discover hidden directories.
* Identify publicly accessible development files.
* Analyze information disclosure.
* Determine whether the disclosed information can be used in subsequent attack phases.

---

## 7.2 Initial Inspection of the Website

### Accessing the Target Website

The web application was opened in a browser using the target IP address.

```text
http://10.10.43.214
```

### Observation

The homepage displayed a simple maintenance message indicating that the website was currently under development.

### Analysis

Although the page appeared minimal, maintenance pages often expose hidden resources that are not linked directly from the homepage.

The absence of visible functionality suggested that further directory enumeration would likely reveal additional content.

---

### 📷 Screenshot Placeholder

**Figure 7.1 — Homepage of Target Web Application**

> Insert a screenshot showing the default Apache maintenance webpage hosted on `10.10.43.214`.

---

## 7.3 Why Web Enumeration Was Performed

Hidden directories are commonly used during development for:

* Testing applications.
* Storing backup files.
* Internal documentation.
* Configuration resources.
* Development notes.

If directory listing or hidden resources are accidentally exposed, attackers can obtain valuable reconnaissance information without exploiting any vulnerability.

For this reason, HTTP enumeration became the next logical step.

---

# 8. HTTP Enumeration Using Nmap NSE

## 8.1 Objective

Identify hidden directories and files exposed by the Apache web server.

Instead of using Gobuster or Dirb, I used Nmap's **HTTP Enumeration NSE Script** because it performs quick discovery of commonly exposed directories and files.

---

## 8.2 Command Used

```bash
sudo nmap -p80 --script http-enum 10.10.43.214
```

---

## 8.3 Command Explanation

| Option               | Description                               |
| -------------------- | ----------------------------------------- |
| `sudo`               | Executes Nmap with elevated privileges.   |
| `-p80`               | Scans only HTTP Port 80.                  |
| `--script http-enum` | Executes the HTTP Enumeration NSE script. |
| `10.10.43.214`       | Target machine IP address.                |

---

## 8.4 Why `http-enum` Was Selected

The `http-enum` NSE script checks the web server against a built-in list of commonly used directories and filenames.

Advantages include:

* Fast reconnaissance.
* No external wordlist required.
* Identifies common admin and development paths.
* Integrates directly into Nmap results.

This approach was sufficient for initial web reconnaissance before performing deeper brute-force enumeration.

---

### 📷 Screenshot Placeholder

**Figure 8.1 — Nmap HTTP Enumeration Scan**

> Insert screenshot of the `http-enum` scan output.

---

## 8.5 Enumeration Results

The HTTP Enumeration script returned an interesting result.

### Discovery

```text
/development/
```

The script reported that this directory was accessible and contained directory listing.

### Analysis

This immediately indicated a potential **information disclosure vulnerability** because development directories are not intended to be publicly accessible.

The discovery suggested that developers may have left internal resources exposed on the web server.

---

## 8.6 Security Observation

The `/development` directory returned a valid HTTP response rather than a **403 Forbidden** or **404 Not Found** response.

This confirmed:

* Directory exists.
* Directory is publicly accessible.
* Directory indexing is enabled.

### Why This Matters

Public directory listing allows attackers to browse internal files without authentication.

This increases the reconnaissance surface significantly.

---

### 📷 Screenshot Placeholder

**Figure 8.2 — Hidden `/development` Directory Identified**

> Insert screenshot showing the `/development` directory discovered by Nmap.

---

# 9. Investigating the Development Directory

## 9.1 Accessing the Directory

The hidden directory was opened in the browser.

```text
http://10.10.43.214/development
```

### Observation

Apache displayed a directory index instead of blocking access.

The directory contained two files.

| File Name | Description                  |
| --------- | ---------------------------- |
| `dev.txt` | Developer notes.             |
| `j.txt`   | Internal communication file. |

---

### 📷 Screenshot Placeholder

**Figure 9.1 — Directory Listing of `/development`**

> Insert screenshot showing both files listed inside the development directory.

---

## 9.2 Analysis of Directory Listing

The existence of directory indexing itself represents an insecure configuration.

### Information Exposed

* File names.
* Modification dates.
* File sizes.
* Internal development artifacts.

### Security Risk

An attacker now knows that developers stored notes inside the web root.

This encouraged inspection of both files individually.

---

# 10. Analysis of `dev.txt`

## 10.1 Objective

Inspect developer notes for sensitive information.

### Access Method

The file was opened directly from the browser.

---

### 📷 Screenshot Placeholder

**Figure 10.1 — Contents of `dev.txt`**

> Insert screenshot of the `dev.txt` contents.

---

## 10.2 Observations

The file contained internal notes written by developers.

Information included:

* References to development activity.
* Mention of application components.
* Technology references.
* Internal comments.

### Important Observation

The file referenced:

* Apache
* REST
* Struts

This indicated the developers were discussing application technologies internally.

---

## 10.3 Security Analysis

Developer notes should never be accessible publicly.

Reasons include:

* Technology stack disclosure.
* Internal architecture hints.
* Potential vulnerability research targets.

### Why This Was Useful

Knowing technologies helps attackers identify:

* Version-specific vulnerabilities.
* Public exploits.
* Misconfigurations.

Although this information did not immediately lead to exploitation, it increased understanding of the application environment.

---

## 10.4 Decision Made

I documented the information but did **not** attempt exploitation immediately.

Instead, I inspected the second file because user-related hints often provide more useful reconnaissance information than technology notes.

---

# 11. Analysis of `j.txt`

## 11.1 Objective

Inspect the second development file for additional intelligence.

The file was opened directly through the browser.

---

### 📷 Screenshot Placeholder

**Figure 11.1 — Contents of `j.txt`**

> Insert screenshot showing the text inside `j.txt`.

---

## 11.2 Observations

The file appeared to contain communication between developers.

Important clues included:

* Mention of an individual identified as **J**.
* Mention of another individual identified as **K**.
* Internal discussion regarding development tasks.

### Analysis

The initials **J** and **K** immediately appeared significant.

Possible interpretations included:

* Usernames.
* Developer accounts.
* System users.
* SSH users.

---

## 11.3 Initial Hypothesis

At this stage, I formed the hypothesis that:

| Initial | Possible Meaning               |
| ------- | ------------------------------ |
| J       | User account beginning with J. |
| K       | User account beginning with K. |

This hypothesis influenced the next stages of enumeration.

---

## 11.4 Security Observation

Even though no credentials were exposed, revealing employee or developer identities reduces attacker effort during authentication attacks.

This is a classic example of **information disclosure**.

---

# 12. Web Enumeration Summary

## Findings Collected

| Finding                   | Security Importance                        |
| ------------------------- | ------------------------------------------ |
| `/development` directory  | Publicly accessible development resources. |
| `dev.txt`                 | Technology disclosure.                     |
| `j.txt`                   | User identity hints.                       |
| Directory Listing Enabled | Information disclosure vulnerability.      |

---

## Security Analysis

The web server leaked valuable reconnaissance information without authentication.

### Information Gained

* Internal development directory.
* Technology stack references.
* User initials.
* Development notes.

### Attack Surface Impact

Although no credentials were discovered during this phase, the exposed information helped narrow future enumeration targets.

---

# 13. Reasoning Behind the Next Decision

At this point, I had collected two possible user initials (**J** and **K**) from the development files.

### Possible Next Options

| Option                                     | Decision                  |
| ------------------------------------------ | ------------------------- |
| Attempt SSH brute force using initials.    | Considered but postponed. |
| Continue SMB enumeration.                  | ✅ Selected.               |
| Search for additional web vulnerabilities. | Deferred.                 |

### Why SMB Enumeration Was Chosen

The Nmap reconnaissance phase already identified SMB services running on ports **139** and **445**.

SMB often exposes:

* Usernames.
* Shared files.
* Anonymous shares.
* Internal documents.

Since the web application hinted at possible usernames, SMB enumeration had a higher probability of revealing complete user information.

Therefore, I transitioned from web enumeration to SMB enumeration.

---
# 📂 Phase 3 – SMB Enumeration & User Discovery

> **Target:** `10.10.43.214`
>
> **Objective:** Enumerate the SMB service to identify shared resources, discover valid users, analyze permissions, and gather intelligence for authenticated access.

---

# 14. SMB Enumeration

## 14.1 Objective

After completing web enumeration, the next attack surface selected was the **Server Message Block (SMB)** service running on the target machine.

The initial Nmap scan identified SMB services listening on **Ports 139 and 445**. SMB is commonly used for file sharing and authentication in both Windows and Linux (Samba) environments. Misconfigured SMB services often expose sensitive information through anonymous shares, user enumeration, or improperly configured permissions.

The goal of this phase was to determine whether SMB exposed any resources that could assist in obtaining valid user information or credentials.

### Objectives of SMB Enumeration

* Identify SMB configuration.
* Discover shared resources.
* Enumerate users.
* Check anonymous access.
* Download publicly accessible files.
* Gather intelligence for authentication attacks.

---

## 14.2 Why SMB Was Investigated

The previous phase revealed possible user initials (**J** and **K**) through developer notes. However, initials alone were insufficient for authentication attacks.

SMB enumeration was chosen because SMB frequently reveals:

* User accounts.
* Shared folders.
* Internal documentation.
* Configuration files.
* Backup files.
* Anonymous resources.

Instead of guessing usernames, SMB enumeration could provide verified user information.

---

# 15. Enumerating SMB with Enum4Linux

## 15.1 About Enum4Linux

**Enum4Linux** is an enumeration utility designed for SMB and Samba services. It automates information gathering against SMB-enabled systems.

The tool collects:

* User accounts.
* Shared folders.
* Password policy information.
* Operating system details.
* NetBIOS information.
* Group information.
* Domain information.

For penetration testers, Enum4Linux is often one of the first tools used after identifying SMB.

---

## 15.2 Command Used

```bash
sudo enum4linux -a 10.10.43.214
```

---

## 15.3 Command Breakdown

| Option         | Purpose                                     |
| -------------- | ------------------------------------------- |
| `sudo`         | Executes the tool with elevated privileges. |
| `enum4linux`   | SMB enumeration utility.                    |
| `-a`           | Performs comprehensive enumeration.         |
| `10.10.43.214` | Target machine IP address.                  |

---

## 15.4 Why the `-a` Flag Was Used

The `-a` option performs multiple enumeration techniques in a single execution.

Instead of running separate scans for:

* Users
* Shares
* Policies
* NetBIOS

the tool automatically performs all available enumeration methods.

This saves time during reconnaissance while maximizing collected information.

---

### 📷 Screenshot Placeholder

**Figure 15.1 — Enum4Linux Complete Enumeration Output**

> Insert screenshot showing the complete Enum4Linux terminal output.

---

# 16. Enum4Linux Results Analysis

The enumeration produced several important pieces of information.

---

## 16.1 NetBIOS Information

### Observation

The SMB service exposed NetBIOS information identifying the host.

### Information Collected

| Information  | Value              |
| ------------ | ------------------ |
| Host Type    | Linux Samba Server |
| NetBIOS Name | BASIC2             |

---

### Analysis

NetBIOS information confirms:

* Target is running Samba.
* SMB service is active.
* Host identification information is publicly available.

Although this information alone is not critical, it contributes to fingerprinting the target.

---

## 16.2 SMB Shares Enumeration

Enum4Linux identified available SMB shares.

### Shares Identified

| Share Name | Purpose                            |
| ---------- | ---------------------------------- |
| Anonymous  | Public file sharing resource.      |
| IPC$       | Inter-process communication share. |

---

### Analysis

The presence of a share named **Anonymous** immediately became the highest-priority finding.

Anonymous shares are often unintentionally exposed and may allow attackers to retrieve files without authentication.

---

### 📷 Screenshot Placeholder

**Figure 16.1 — SMB Shares Identified**

> Insert screenshot highlighting the Anonymous share in Enum4Linux output.

---

## 16.3 User Enumeration Results

Enum4Linux successfully enumerated user accounts present on the system.

### Users Identified

* jan
* kay
* ubuntu

---

### Analysis

This finding confirmed the hypothesis generated during web enumeration.

Previously:

| Source    | Information |
| --------- | ----------- |
| `j.txt`   | Initial "J" |
| `dev.txt` | Initial "K" |

Now the SMB service revealed the complete usernames:

| Initial | Username |
| ------- | -------- |
| J       | jan      |
| K       | kay      |

This eliminated guesswork for future authentication attempts.

---

### 📷 Screenshot Placeholder

**Figure 16.2 — User Enumeration Results**

> Insert screenshot showing usernames discovered by Enum4Linux.

---

## 16.4 Security Observation

The SMB service exposed valid usernames without requiring authentication.

### Security Impact

Username enumeration significantly increases the success probability of:

* Password attacks.
* Credential stuffing.
* Social engineering.
* Brute-force attacks.

This represents an information disclosure vulnerability.

---

# 17. Manual SMB Share Enumeration

## 17.1 Why Manual Enumeration Was Performed

Although Enum4Linux identified the anonymous share, I manually accessed the share to inspect its contents.

Manual enumeration helps verify:

* Read permissions.
* Write permissions.
* Available files.
* Share accessibility.

---

## 17.2 Accessing SMB Share

### Command Used

```bash
smbclient //10.10.43.214/Anonymous
```

---

## 17.3 Command Explanation

| Component        | Description              |
| ---------------- | ------------------------ |
| `smbclient`      | SMB client utility.      |
| `//10.10.43.214` | Target SMB server.       |
| `/Anonymous`     | Anonymous shared folder. |

---

### Observation

The connection succeeded without requiring valid credentials.

This confirmed anonymous access was enabled.

---

### 📷 Screenshot Placeholder

**Figure 17.1 — Successful Anonymous SMB Login**

> Insert screenshot showing successful connection to the Anonymous share.

---

# 18. Inspecting Files Inside the SMB Share

## 18.1 Listing Available Files

After connecting to the share, I listed available files.

### Command Used

```bash
dir
```

---

### Observation

The share contained one interesting file.

| File        | Purpose                    |
| ----------- | -------------------------- |
| `staff.txt` | Staff-related information. |

---

### Analysis

A file named `staff.txt` immediately appeared valuable because it suggested employee or user information.

This became the primary target for download.

---

### 📷 Screenshot Placeholder

**Figure 18.1 — Contents of Anonymous SMB Share**

> Insert screenshot showing the output of the `dir` command.

---

## 18.2 Downloading the File

### Command Used

```bash
get staff.txt
```

---

### Command Explanation

| Command     | Purpose                                                |
| ----------- | ------------------------------------------------------ |
| `get`       | Downloads a file from SMB share to local Kali machine. |
| `staff.txt` | Target file.                                           |

---

### Observation

The download completed successfully.

The file was stored locally for analysis.

---

### 📷 Screenshot Placeholder

**Figure 18.2 — Downloading `staff.txt`**

> Insert screenshot showing successful execution of the `get` command.

---

# 19. Analysis of `staff.txt`

## 19.1 Opening the File

The downloaded file was inspected locally.

### Observation

The file contained names of staff members.

### Information Found

* jan
* kay

---

### Analysis

This finding confirmed the usernames obtained from Enum4Linux.

The information was now verified from two independent sources:

| Source     | Information      |
| ---------- | ---------------- |
| Enum4Linux | User Enumeration |
| staff.txt  | Staff Usernames  |

---

## 19.2 Why This Was Important

Password attacks require **both**:

* Valid usernames.
* Password candidates.

Before SMB enumeration, only initials were known.

After SMB enumeration:

* Valid usernames were confirmed.
* Authentication attacks became much more targeted.

---

### Security Impact

Anonymous access exposed internal organizational user information.

Even without passwords, usernames dramatically reduce attack complexity.

---

### 📷 Screenshot Placeholder

**Figure 19.1 — Contents of `staff.txt`**

> Insert screenshot showing usernames inside `staff.txt`.

---

# 20. Comparing Enumeration Results

At this stage, information gathered from different attack surfaces was correlated.

| Enumeration Source | Intelligence Obtained               |
| ------------------ | ----------------------------------- |
| Nmap               | SSH, HTTP, SMB services identified. |
| HTTP Enumeration   | Developer notes and user initials.  |
| Enum4Linux         | Valid Linux usernames.              |
| SMB Share          | Staff usernames confirmed.          |

---

## Intelligence Correlation

```text
Web Enumeration
      │
      ▼
J and K Identified
      │
      ▼
SMB Enumeration
      │
      ▼
jan and kay Confirmed
      │
      ▼
Valid Authentication Targets Identified
```

This demonstrates how multiple reconnaissance techniques combine to build an attack path.

---

# 21. Why SSH Became the Next Target

## 21.1 Available Attack Surfaces Revisited

After SMB enumeration, the target exposed:

| Service | Status                    |
| ------- | ------------------------- |
| HTTP    | Information gathered.     |
| SMB     | Usernames gathered.       |
| SSH     | Authentication available. |

---

## 21.2 Decision Analysis

Possible next actions included:

### Option A — Continue SMB Enumeration

Pros:

* More shares.

Cons:

* No additional sensitive resources discovered.

### Option B — Exploit HTTP

Pros:

* Possible application vulnerability.

Cons:

* No immediate exploit confirmed.

### Option C — Authenticate via SSH

Pros:

* Valid usernames available.
* SSH service exposed.
* Password attack possible.

### Decision

**SSH authentication attack** became the most efficient next step.

---

## Why Hydra Was Selected

Hydra supports online authentication attacks against SSH using username and password dictionaries.

Since valid usernames were now known, Hydra could test passwords systematically using a commonly used wordlist.

---

# 22. Security Analysis of SMB Phase

## Vulnerabilities Identified

| Finding                            | Severity |
| ---------------------------------- | -------- |
| Anonymous SMB Share Enabled        | High     |
| Username Enumeration Allowed       | High     |
| Internal Staff Information Exposed | High     |

---

## Attack Chain Contribution

This phase was critical because it provided authenticated attack targets.

Without SMB enumeration:

* SSH brute force would require guessing usernames.
* Authentication attempts would be significantly less efficient.

Instead, SMB reduced uncertainty and directly enabled the next penetration testing phase.

---

# 🔐 Phase 4 – SSH Credential Discovery & Initial Access

> **Target:** `10.10.43.214`
>
> **Objective:** Use the usernames discovered during SMB enumeration to perform an authorized SSH password attack, obtain authenticated access to the target machine, and begin post-authentication enumeration.

---

# 24. SSH Authentication Assessment

## 24.1 Objective

After completing SMB enumeration, I had confirmed two valid usernames (`jan` and `kay`). The next objective was to determine whether these accounts were protected by weak passwords.

The target machine exposed **SSH** on Port **22**, making it a potential authentication entry point.

Instead of manually testing passwords, I used **Hydra**, an online password auditing tool, to perform a dictionary attack against the SSH service.

### Objectives of this Phase

* Verify SSH authentication.
* Test discovered usernames.
* Perform an authorized dictionary attack.
* Obtain initial shell access.
* Begin authenticated Linux enumeration.

---

## 24.2 Why Hydra Was Selected

Hydra is a penetration testing tool used for **online authentication testing** against multiple network protocols, including SSH.

It was selected because:

* SSH authentication was enabled.
* Valid usernames had already been discovered.
* The assessment was conducted inside an authorized lab environment.
* Hydra supports dictionary attacks using common password lists.

### Attack Method Used

**Dictionary Attack**

A dictionary attack tests passwords from a predefined wordlist rather than generating random passwords.

Advantages include:

* Faster than brute force.
* Common weak passwords are detected quickly.
* Widely used during penetration testing to evaluate password strength.

---

# 25. Performing the SSH Dictionary Attack

## 25.1 Command Used

```bash
hydra -l jan -P /usr/share/wordlists/rockyou.txt ssh://10.10.43.214 -I
```

---

## 25.2 Command Breakdown

| Parameter            | Description                                             |
| -------------------- | ------------------------------------------------------- |
| `hydra`              | Password auditing tool.                                 |
| `-l jan`             | Specifies a single username (`jan`).                    |
| `-P`                 | Specifies a password wordlist.                          |
| `rockyou.txt`        | Popular dictionary containing millions of passwords.    |
| `ssh://10.10.43.214` | Target SSH service.                                     |
| `-I`                 | Ignore previous Hydra restore sessions and start fresh. |

---

## 25.3 Why RockYou Wordlist Was Used

The **RockYou** wordlist is included in Kali Linux and contains millions of passwords collected from historical password leaks.

It is commonly used during password security assessments because it contains:

* Weak passwords.
* Common passwords.
* Frequently reused passwords.
* Dictionary words.

This makes it effective for evaluating password policy weaknesses.

---

### 📷 Screenshot Placeholder

**Figure 25.1 — Hydra Dictionary Attack Execution**

> Insert screenshot showing the Hydra command executing against `10.10.43.214`.

---

# 26. Hydra Attack Results

## 26.1 Observation

Hydra attempted passwords from the dictionary against the `jan` account.

After testing multiple entries, Hydra reported a successful authentication.

### Result

A valid SSH credential was identified for the user **jan**.

> **Note:** In the GitHub portfolio, avoid publishing recovered passwords. State only that valid credentials were obtained during the authorized assessment.

---

### 📷 Screenshot Placeholder

**Figure 26.1 — Hydra Successfully Identifies Valid SSH Credentials**

> Insert screenshot highlighting Hydra's success message.

---

## 26.2 Analysis of Results

The successful dictionary attack indicates that the account was protected by a password present in a commonly used password dictionary.

### Security Finding

**Weak Password Policy**

### Why This Is Important

A weak password increases the likelihood of unauthorized access through:

* Dictionary attacks.
* Password spraying.
* Credential stuffing.
* Automated authentication attacks.

### Risk Assessment

| Factor                 | Analysis                         |
| ---------------------- | -------------------------------- |
| Authentication Service | SSH                              |
| Attack Type            | Dictionary Attack                |
| Result                 | Successful Authentication        |
| Security Impact        | Unauthorized Remote Shell Access |

---

## 26.3 Why the Attack Was Not Continued Against `kay`

During the assessment, the `kay` username was also identified.

However:

* Hydra successfully authenticated `jan`.
* `kay` did not immediately produce valid credentials.
* Continuing the attack was unnecessary because authenticated access had already been achieved.

### Decision

Instead of spending additional time brute-forcing another account, I proceeded with the authenticated account (`jan`) to perform local enumeration.

This follows the penetration testing principle of **using the least effort to gain maximum information**.

---

# 27. Initial SSH Access

## 27.1 Objective

Authenticate into the target machine using the credentials obtained during the authorized password assessment.

---

## 27.2 SSH Login Command

```bash
ssh jan@10.10.43.214
```

---

## 27.3 Command Explanation

| Component      | Description               |
| -------------- | ------------------------- |
| `ssh`          | Secure Shell client.      |
| `jan`          | Authenticated Linux user. |
| `10.10.43.214` | Target machine IP.        |

---

### Observation

The SSH connection was established successfully.

After entering the recovered password, an interactive Linux shell was opened.

This marked the completion of the **Initial Access** phase.

---

### 📷 Screenshot Placeholder

**Figure 27.1 — Successful SSH Login as `jan`**

> Insert screenshot showing the terminal prompt after logging into the target machine.

---

# 28. Post-Authentication Enumeration Begins

## 28.1 Objective

Once authenticated access was obtained, the focus shifted from external reconnaissance to **local enumeration**.

Local enumeration is essential because authenticated users often have access to additional information unavailable externally.

### Objectives

* Identify additional users.
* Enumerate file permissions.
* Locate sensitive files.
* Discover privilege escalation vectors.

---

## 28.2 Verify Current User

### Command Used

```bash
whoami
```

### Observation

Output confirmed the current authenticated user was:

```text
jan
```

---

### Analysis

This verification ensured that all subsequent enumeration activities were performed using the permissions assigned to the `jan` account.

---

### 📷 Screenshot Placeholder

**Figure 28.1 — Verifying Current User**

> Insert screenshot showing output of `whoami`.

---

## 28.3 Identify User ID and Groups

### Command Used

```bash
id
```

### Purpose

Displays:

* User ID (UID)
* Group ID (GID)
* Group memberships

### Analysis

This information helps determine whether the current account belongs to privileged groups such as:

* sudo
* admin
* docker
* lxd

### Observation

The account had standard user privileges.

No administrative privileges were immediately available.

---

### 📷 Screenshot Placeholder

**Figure 28.2 — User Identity Information**

> Insert screenshot showing output of `id`.

---

# 29. Enumerating Linux User Accounts

## 29.1 Objective

Identify all user accounts present on the Linux system.

### Command Used

```bash
cat /etc/passwd
```

---

## 29.2 Why `/etc/passwd` Was Inspected

`/etc/passwd` stores information about every user account on a Linux system.

It contains:

* Username.
* User ID.
* Group ID.
* Home directory.
* Default shell.

### Importance During Penetration Testing

Helps identify:

* Human users.
* Service accounts.
* Home directories.
* Potential privilege escalation targets.

---

### 📷 Screenshot Placeholder

**Figure 29.1 — Contents of `/etc/passwd`**

> Insert screenshot showing the `/etc/passwd` output.

---

## 29.3 Important User Accounts Identified

Among many system accounts, three users stood out.

| Username | Observation                    |
| -------- | ------------------------------ |
| `jan`    | Current authenticated user.    |
| `kay`    | Additional human user account. |
| `ubuntu` | Another local user account.    |

---

## 29.4 Analysis of User Enumeration

The `kay` account immediately became interesting because:

* It was previously identified through SMB.
* It appeared as a valid Linux user.
* It possessed its own home directory.

This suggested that `kay` could contain files inaccessible to `jan`.

### Security Observation

Authenticated user enumeration often reveals additional targets for privilege escalation.

---

# 30. Investigating Home Directories

## 30.1 Objective

Inspect local user home directories for sensitive resources.

### Navigate to Home Directory

```bash
cd /home
```

### List Available Users

```bash
ls
```

---

### Observation

The `/home` directory contained:

```text
jan
kay
ubuntu
```

---

### Analysis

This confirmed that multiple human users existed on the system.

The next logical step was to inspect the `kay` home directory because it belonged to another user.

---

### 📷 Screenshot Placeholder

**Figure 30.1 — Listing `/home` Directory**

> Insert screenshot showing available user directories.

---

## 30.2 Navigate to `kay` Directory

### Command Used

```bash
cd /home/kay
```

### Observation

The directory was accessible for listing.

### Analysis

Being able to enter another user's home directory does **not** necessarily imply permission to read all files.

The next step was permission enumeration.

---

# 31. Enumerating Files and Permissions

## 31.1 List Hidden Files

### Command Used

```bash
ls -la
```

---

## 31.2 Why `ls -la` Was Used

Unlike `ls`, the `-la` flags display:

* Hidden files.
* File permissions.
* Ownership.
* Groups.
* Sizes.
* Modification timestamps.

This command is fundamental during Linux privilege escalation.

---

### 📷 Screenshot Placeholder

**Figure 31.1 — File Permission Enumeration in `/home/kay`**

> Insert screenshot showing `ls -la` output.

---

## 31.3 Interesting Files Identified

| File            | Initial Observation          |
| --------------- | ---------------------------- |
| `pass.bak`      | Restricted backup file.      |
| `.ssh/`         | SSH configuration directory. |
| `.bash_history` | Command history.             |
| `.cache`        | User cache directory.        |

---

### Analysis

Two resources immediately stood out:

1. `pass.bak`
2. `.ssh`

These commonly contain authentication-related information.

---

# 32. Attempting to Read `pass.bak`

## 32.1 Command Used

```bash
cat pass.bak
```

### Observation

The command returned:

```text
Permission denied
```

---

### Analysis

This indicated:

* File existed.
* File was readable only by `kay`.
* Current user (`jan`) lacked permissions.

### Security Interpretation

This suggested the file contained sensitive information worth targeting during privilege escalation.

---

### 📷 Screenshot Placeholder

**Figure 32.1 — Permission Denied While Reading `pass.bak`**

> Insert screenshot showing the permission denied message.

---

# 33. File Permission Analysis

## 33.1 Understanding Permissions

`ls -la` showed restrictive permissions for `pass.bak`.

Example format:

```text
-rw-------
```

### Permission Breakdown

| Permission | Meaning      |
| ---------- | ------------ |
| Owner      | Read + Write |
| Group      | No Access    |
| Others     | No Access    |

### Analysis

Only the owner (`kay`) could read the file.

The current account (`jan`) could not.

---

## 33.2 Why This Was Important

The inability to read `pass.bak` indicated that **privilege escalation** would be required to access its contents.

Rather than abandoning the file, I continued enumerating the `.ssh` directory for alternative access methods.

---

# 34. Analysis After Initial Access Phase

## Information Collected So Far

| Source        | Information Obtained        |
| ------------- | --------------------------- |
| Hydra         | Valid SSH authentication.   |
| SSH           | Interactive shell access.   |
| `/etc/passwd` | Additional user (`kay`).    |
| `/home`       | Multiple home directories.  |
| `ls -la`      | Sensitive files identified. |
| `pass.bak`    | Access restricted.          |

---

## Why `.ssh` Became the Next Target

The `.ssh` directory often contains:

* SSH private keys.
* Public keys.
* Authorized keys.
* Known hosts.

If misconfigured, it may expose authentication material that allows access to another account.

Since `pass.bak` was inaccessible, inspecting `.ssh` became the next logical privilege escalation path.

---

# ⬆️ Phase 5 – Linux Enumeration & Privilege Escalation Discovery

> **Target:** `10.10.43.214`
>
> **Objective:** Perform detailed Linux enumeration after obtaining SSH access as the user `jan`, identify privilege escalation opportunities, analyze SSH configuration files, and discover sensitive authentication material belonging to another user.

---

# 35. Linux Privilege Escalation Enumeration

## 35.1 Objective

After gaining authenticated SSH access as the user **jan**, the penetration testing process entered the **post-exploitation enumeration** phase.

The purpose of this stage was **not** to immediately escalate privileges, but to systematically inspect the Linux environment for misconfigurations, sensitive files, credentials, or authentication mechanisms that could provide access to higher-privileged users.

Professional penetration testing engagements always perform comprehensive local enumeration before attempting privilege escalation.

### Objectives of This Phase

* Enumerate user home directories.
* Inspect hidden files.
* Identify SSH authentication files.
* Analyze Linux file permissions.
* Discover potential privilege escalation vectors.
* Preserve sensitive authentication material for offline analysis.

---

## 35.2 Why Local Enumeration Is Important

Once authenticated access is obtained, the attacker's visibility changes dramatically.

Information available after login includes:

* Local users.
* Home directories.
* Hidden files.
* SSH configuration.
* Authentication material.
* Backup files.
* Scheduled tasks.
* Environment variables.

This information is unavailable during external reconnaissance.

The objective is to convert a **low-privileged user** into a **higher-privileged user** through legitimate security weaknesses.

---

# 36. Inspecting the Target User's Home Directory

## 36.1 Navigate to the User Directory

The first step was confirming the current working directory and inspecting accessible files.

### Command Used

```bash
pwd
```

### Observation

The command displayed the current directory associated with the authenticated user.

### Analysis

This confirms that the SSH session started inside the home directory of `jan`.

---

### 📷 Screenshot Placeholder

**Figure 36.1 — Current Working Directory**

> Insert screenshot showing the output of `pwd`.

---

## 36.2 Listing Files in the Home Directory

### Command Used

```bash
ls -la
```

### Why `ls -la` Was Used

The `-la` flags display:

* Hidden files.
* File permissions.
* Ownership.
* Groups.
* Timestamps.
* Directory contents.

Hidden files frequently contain authentication or configuration information.

### Observation

The home directory contained standard Linux configuration files.

Examples included:

| Hidden File     | Purpose                          |
| --------------- | -------------------------------- |
| `.bash_history` | Shell command history.           |
| `.bashrc`       | User shell configuration.        |
| `.profile`      | Login environment configuration. |

### Analysis

No obvious privilege escalation vector existed inside the current user's home directory.

The investigation therefore shifted toward another user's directory.

---

### 📷 Screenshot Placeholder

**Figure 36.2 — Listing Hidden Files in User Home Directory**

> Insert screenshot showing `ls -la` output.

---

# 37. Investigating the `kay` User Directory

## 37.1 Why the `kay` Directory Was Investigated

During the previous phase, enumeration revealed another user named **kay**.

Reasons this account became interesting:

* Previously discovered through SMB.
* Confirmed inside `/etc/passwd`.
* Separate home directory existed.
* Restricted backup file (`pass.bak`) belonged to this account.

The goal was to inspect accessible files without modifying the system.

---

## 37.2 Navigating to `/home/kay`

### Command Used

```bash
cd /home/kay
```

### Observation

The directory was accessible for listing, even though individual files were protected.

### Analysis

Linux permissions allow directory traversal independently of file read permissions.

This meant metadata could still be inspected.

---

### 📷 Screenshot Placeholder

**Figure 37.1 — Navigating to `/home/kay`**

> Insert screenshot showing successful directory change.

---

# 38. Enumerating Files Inside `/home/kay`

## 38.1 Listing Hidden Files

### Command Used

```bash
ls -la
```

### Observation

Several interesting files and directories were identified.

| File / Directory | Observation                  |
| ---------------- | ---------------------------- |
| `pass.bak`       | Backup password file.        |
| `.ssh/`          | SSH configuration directory. |
| `.bash_history`  | Shell history.               |
| `.cache/`        | User cache directory.        |

---

### Analysis

The `.ssh` directory immediately became the highest-priority target because SSH authentication files frequently contain sensitive credentials.

---

### 📷 Screenshot Placeholder

**Figure 38.1 — Enumeration of `/home/kay`**

> Insert screenshot showing all files listed.

---

# 39. Understanding the `.ssh` Directory

## 39.1 Why `.ssh` Is Important

Every Linux user's `.ssh` directory stores SSH authentication configuration.

Common files include:

| File              | Purpose                           |
| ----------------- | --------------------------------- |
| `authorized_keys` | Public keys allowed for login.    |
| `id_rsa`          | Private SSH key.                  |
| `id_rsa.pub`      | Public SSH key.                   |
| `known_hosts`     | Previously connected SSH servers. |

These files determine how SSH authentication is performed.

---

## 39.2 Listing `.ssh` Contents

### Command Used

```bash
ls -la /home/kay/.ssh
```

### Observation

The directory contained authentication-related files.

### Files Identified

| File              | Description             |
| ----------------- | ----------------------- |
| `authorized_keys` | Authorized public keys. |
| `id_rsa`          | Private SSH key.        |
| `id_rsa.pub`      | Public SSH key.         |

---

### Analysis

Finding a private SSH key inside another user's directory is highly significant.

However, the key was encrypted with a passphrase.

This suggested an opportunity for **offline password cracking** instead of online authentication attacks.

---

### 📷 Screenshot Placeholder

**Figure 39.1 — Contents of `.ssh` Directory**

> Insert screenshot showing the files inside `.ssh`.

---

# 40. Understanding SSH Authentication Files

## 40.1 `authorized_keys`

### Purpose

Stores public keys allowed to authenticate into the account.

### Security Significance

Anyone possessing the matching private key can authenticate without a password.

---

## 40.2 `id_rsa`

### Purpose

Private SSH authentication key.

### Security Importance

This file must remain secret.

If exposed:

* Password authentication becomes unnecessary.
* Offline attacks become possible.
* User impersonation becomes possible after recovering the passphrase.

---

## 40.3 `id_rsa.pub`

### Purpose

Public component of the SSH key pair.

### Security Observation

Public keys are intended to be shared.

They do not provide authentication by themselves.

---

# 41. Inspecting File Permissions

## 41.1 Checking Permissions

The permissions displayed by `ls -la` were analyzed.

### Observation

The private key existed and could be copied from the user's directory.

### Security Analysis

The exposure of another user's private key indicates improper permission management.

Even if encrypted, attackers can perform offline attacks against the passphrase.

---

### 📷 Screenshot Placeholder

**Figure 41.1 — File Permissions of SSH Keys**

> Insert screenshot highlighting `id_rsa`.

---

# 42. Accessing the Private Key

## 42.1 Viewing the Private Key

### Command Used

```bash
cat /home/kay/.ssh/id_rsa
```

### Observation

The output displayed an encrypted OpenSSH private key beginning with the standard header.

### Analysis

The file was encrypted, meaning direct SSH authentication was not yet possible.

Instead, the private key needed to be copied locally for offline analysis.

---

### 📷 Screenshot Placeholder

**Figure 42.1 — Encrypted OpenSSH Private Key**

> Insert screenshot showing only the beginning of the private key header (avoid exposing the full key in GitHub).

---

## 42.2 Why the Private Key Was Not Modified

Professional penetration testing practice requires preserving evidence.

Instead of editing the original key:

* A local copy was created.
* The original file remained unchanged.
* Offline cracking was performed from Kali Linux.

This minimizes changes on the target machine.

---

# 43. Copying the SSH Private Key

## 43.1 Saving the Key Locally

The encrypted private key was copied from the target system into a local file on Kali Linux.

### Local Filename Used

```text
bp
```

### Purpose

This file served as the input for offline password cracking.

---

### 📷 Screenshot Placeholder

**Figure 43.1 — Private Key Saved Locally**

> Insert screenshot showing the local file containing the SSH private key.

---

## 43.2 Restricting File Permissions

### Command Used

```bash
chmod 400 bp
```

### Command Explanation

| Permission | Meaning                    |
| ---------- | -------------------------- |
| `4`        | Read permission.           |
| `0`        | No permissions for group.  |
| `0`        | No permissions for others. |

### Why This Was Necessary

SSH refuses to use private keys with overly permissive permissions.

Setting the permission to `400` ensures:

* Owner can read.
* Nobody else can access the file.
* SSH accepts the key later during authentication.

---

### 📷 Screenshot Placeholder

**Figure 43.2 — Restricting Private Key Permissions**

> Insert screenshot showing successful execution of `chmod 400 bp`.

---

# 44. Verifying the Local Key File

## 44.1 Listing Permissions

### Command Used

```bash
ls -l bp
```

### Observation

The file displayed read-only permissions for the owner.

### Analysis

The key was now properly prepared for offline processing.

---

### 📷 Screenshot Placeholder

**Figure 44.1 — Verification of Local Private Key Permissions**

> Insert screenshot showing the permission output for `bp`.

---

# 45. Privilege Escalation Analysis

## 45.1 Enumeration Findings

The local enumeration phase produced several critical findings.

| Finding                         | Importance                         |
| ------------------------------- | ---------------------------------- |
| Additional user account (`kay`) | Privilege escalation target.       |
| `pass.bak`                      | Sensitive restricted file.         |
| `.ssh` directory                | Authentication files discovered.   |
| `id_rsa`                        | Encrypted private SSH key exposed. |

---

## 45.2 Why the SSH Key Was the Best Attack Vector

Several privilege escalation possibilities existed.

### Option A — Crack Linux Password

No password hashes were available.

### Option B — Read Restricted Files

Permission denied.

### Option C — Use SSH Private Key

Private key available for offline analysis.

### Decision

**Offline SSH key passphrase cracking** became the selected privilege escalation technique.

This approach avoids repeated authentication attempts against the server and performs password recovery locally.

---

# 46. Preparing for Offline Password Cracking

## Objective of the Next Phase

The private SSH key was encrypted.

Before it could be used for authentication, its passphrase needed to be recovered.

The next phase involves:

1. Locating the `ssh2john.py` utility.
2. Converting the SSH private key into a John-the-Ripper-compatible hash.
3. Performing an offline dictionary attack.
4. Recovering the passphrase.
5. Authenticating as the user `kay`.

---

# 🔓 Phase 6 – SSH Private Key Cracking & Privilege Escalation

> **Target:** `10.10.43.214`
>
> **Objective:** Convert the encrypted SSH private key into a crackable hash, recover its passphrase using an offline dictionary attack, and authenticate as the user `kay` to achieve privilege escalation.

---

# 47. Beginning the Privilege Escalation Phase

## 47.1 Objective

The previous enumeration phase revealed an encrypted SSH private key (`id_rsa`) belonging to the user **kay**. Although the key was protected with a passphrase, it represented a valuable privilege escalation vector because password recovery could be performed **offline** without generating repeated authentication attempts against the target machine.

The goal of this phase was to:

* Prepare the SSH private key for offline analysis.
* Convert it into a format understood by John the Ripper.
* Recover the passphrase using a dictionary attack.
* Authenticate into the system as `kay`.
* Gain access to resources unavailable to the `jan` account.

---

## 47.2 Why Offline Cracking Was Selected

There were two possible approaches after obtaining the private key.

### Option A — Attempt Online Authentication

This would repeatedly contact the SSH server with guessed passphrases.

**Disadvantages**

* Generates authentication logs.
* Easier to detect.
* Slower.

### Option B — Offline Password Cracking

Uses the encrypted private key locally.

**Advantages**

* No interaction with target server.
* Unlimited password attempts locally.
* Faster using optimized cracking tools.
* No additional network traffic.

### Decision

Offline cracking was selected because it is the preferred penetration testing technique when encrypted authentication material has already been obtained.

---

# 48. Preparing the SSH Private Key

## 48.1 Verify the Key File

Before cracking the key, I verified that the copied private key existed on the Kali Linux machine.

### Command Used

```bash id="6ssqis"
ls -l bp
```

### Observation

The file `bp` existed with restricted read permissions (`400`).

### Analysis

This confirmed that:

* The key had been copied successfully.
* SSH-compatible permissions had been applied.
* The file was ready for offline processing.

---

### 📷 Screenshot Placeholder

**Figure 48.1 — Verification of Local SSH Private Key**

> Insert screenshot showing `ls -l bp`.

---

## 48.2 Inspecting the Key Header

### Command Used

```bash id="h2wlkj"
head -5 bp
```

### Purpose

Display only the beginning of the private key.

### Observation

The key began with the standard OpenSSH header.

### Analysis

The header confirmed:

* OpenSSH private key format.
* Encrypted key file.
* Compatible with `ssh2john.py`.

For security reasons, only the header should be documented in GitHub rather than the complete key contents.

---

### 📷 Screenshot Placeholder

**Figure 48.2 — OpenSSH Private Key Header**

> Insert screenshot showing only the first few lines of the key.

---

# 49. Locating the SSH Key Conversion Utility

## 49.1 Why `ssh2john.py` Is Required

John the Ripper cannot directly process encrypted OpenSSH private keys.

The key must first be converted into a **password hash representation** that John understands.

The conversion utility provided with John the Ripper performs this transformation.

---

## 49.2 Locate the Script

### Command Used

```bash id="tluqj9"
locate ssh2john.py
```

### Observation

The system returned the installation path of `ssh2john.py`.

### Analysis

This confirmed that John the Ripper was installed correctly on Kali Linux and included the conversion utility.

---

### 📷 Screenshot Placeholder

**Figure 49.1 — Locating `ssh2john.py`**

> Insert screenshot showing the path returned by `locate`.

---

# 50. Converting the SSH Key into a John-Compatible Hash

## 50.1 Objective

Transform the encrypted SSH private key into a password hash that John the Ripper can attack.

---

## 50.2 Conversion Command

### Command Used

```bash id="pznjp4"
python3 /usr/share/john/ssh2john.py bp > hash.txt
```

---

## 50.3 Command Breakdown

| Component     | Description                                |
| ------------- | ------------------------------------------ |
| `python3`     | Executes the conversion script.            |
| `ssh2john.py` | Converts OpenSSH private keys into hashes. |
| `bp`          | Input private key file.                    |
| `>`           | Redirect output into a new file.           |
| `hash.txt`    | Output hash file.                          |

---

## 50.4 Why Output Was Redirected

Instead of printing the hash to the terminal, it was saved into `hash.txt`.

Advantages:

* Cleaner workflow.
* Compatible with John.
* Preserves converted hash for reuse.

---

### 📷 Screenshot Placeholder

**Figure 50.1 — SSH Key Converted into Hash Format**

> Insert screenshot showing successful execution of `ssh2john.py`.

---

## 50.5 Verifying the Hash File

### Command Used

```bash id="l1l4vu"
cat hash.txt
```

### Observation

A long hash representing the encrypted SSH key was generated.

### Analysis

The conversion was successful.

This hash contained the encrypted information John needed to recover the passphrase.

---

### 📷 Screenshot Placeholder

**Figure 50.2 — Generated Hash File**

> Insert screenshot showing the generated hash (avoid exposing the complete hash if publishing publicly).

---

# 51. Password Cracking with John the Ripper

## 51.1 About John the Ripper

John the Ripper is an offline password auditing tool capable of cracking:

* Password hashes.
* ZIP passwords.
* SSH keys.
* Linux hashes.
* Windows hashes.
* Database hashes.

It compares candidate passwords against encrypted authentication material until a match is found.

---

## 51.2 Why John Was Used

Hydra attacks live services.

John attacks **offline encrypted files**.

Since the SSH key was already copied locally, John became the correct tool.

---

# 52. Launching the Dictionary Attack

## 52.1 Command Used

```bash id="hjk6xp"
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

---

## 52.2 Command Explanation

| Option        | Purpose                            |
| ------------- | ---------------------------------- |
| `john`        | Password cracking tool.            |
| `--wordlist`  | Specifies password dictionary.     |
| `rockyou.txt` | Password wordlist.                 |
| `hash.txt`    | SSH key hash generated previously. |

---

## 52.3 Why the Same Wordlist Was Used

The RockYou dictionary is widely used during penetration testing because it contains millions of real passwords collected from historical password breaches.

Using the same dictionary allows testing whether the SSH passphrase is also weak.

---

### 📷 Screenshot Placeholder

**Figure 52.1 — John the Ripper Dictionary Attack**

> Insert screenshot showing John beginning the cracking process.

---

# 53. John the Ripper Results

## 53.1 Observation

John processed the hash and successfully recovered the SSH key passphrase.

### Result

A valid passphrase protecting the encrypted private key was recovered.

> **Portfolio Note:** Do not publish the recovered passphrase in GitHub. State only that a valid passphrase was successfully recovered during the authorized assessment.

---

### 📷 Screenshot Placeholder

**Figure 53.1 — Successful Passphrase Recovery**

> Insert screenshot highlighting John's success message.

---

## 53.2 Analysis of the Result

The successful dictionary attack demonstrated that the private key was protected by a **weak passphrase**.

### Security Finding

**Weak SSH Key Passphrase**

### Why This Is Critical

SSH keys are considered stronger than passwords only when their passphrases are also strong.

If an attacker gains access to an encrypted private key and its passphrase is weak, the key effectively becomes compromised.

---

## 53.3 Displaying Stored Results

### Command Used

```bash id="nkqzly"
john --show hash.txt
```

### Purpose

Displays recovered credentials stored inside John's session database.

### Observation

The recovered passphrase was displayed successfully.

### Analysis

This verified that the cracking process completed successfully and the passphrase was stored locally.

---

### 📷 Screenshot Placeholder

**Figure 53.2 — Displaying Cracked Passphrase**

> Insert screenshot showing the output of `john --show`.

---

# 54. Authenticating as the User `kay`

## 54.1 Objective

Use the recovered private key and passphrase to authenticate as the second user.

This represents the actual **privilege escalation step**.

---

## 54.2 SSH Login Using the Private Key

### Command Used

```bash id="dgwfyg"
ssh -i bp kay@10.10.43.214
```

---

## 54.3 Command Breakdown

| Component      | Description                     |
| -------------- | ------------------------------- |
| `ssh`          | Secure Shell client.            |
| `-i bp`        | Specifies the private key file. |
| `kay`          | Target Linux user.              |
| `10.10.43.214` | Target machine IP.              |

---

## 54.4 Authentication Process

SSH requested the private key passphrase.

The recovered passphrase was entered.

Authentication completed successfully.

---

### Observation

A new shell session opened under the account **kay**.

---

### 📷 Screenshot Placeholder

**Figure 54.1 — Successful SSH Authentication Using Private Key**

> Insert screenshot showing login as `kay`.

---

# 55. Verifying Privilege Escalation

## 55.1 Confirm Current User

### Command Used

```bash id="wuxn5u"
whoami
```

### Observation

Output returned:

```text id="k3mdc5"
kay
```

---

### Analysis

This confirmed privilege escalation was successful.

The active shell now had the permissions assigned to `kay`, allowing access to files previously restricted from `jan`.

---

### 📷 Screenshot Placeholder

**Figure 55.1 — Verifying Current User (`kay`)**

> Insert screenshot showing `whoami`.

---

## 55.2 Confirm User Identity

### Command Used

```bash id="lf5tqk"
id
```

### Purpose

Display user ID and group memberships.

### Observation

Output confirmed the shell belonged to the `kay` account.

### Analysis

This verified that authentication occurred through the recovered SSH key rather than password reuse.

---

### 📷 Screenshot Placeholder

**Figure 55.2 — User Identity After Privilege Escalation**

> Insert screenshot showing `id`.

---

# 56. Why This Was Considered Privilege Escalation

## 56.1 Initial Access vs Privilege Escalation

| Stage                | User  |
| -------------------- | ----- |
| Initial Access       | `jan` |
| Privilege Escalation | `kay` |

The assessment moved from one authenticated user to another with access to additional resources.

---

## 56.2 Security Significance

The attack succeeded because of multiple chained weaknesses:

1. Username disclosure.
2. Weak SSH password.
3. Exposed private key.
4. Weak SSH key passphrase.

Each vulnerability alone was concerning.

Combined together, they enabled privilege escalation.

---

# 57. Attack Chain Analysis

## Privilege Escalation Flow

```text id="2n9zzk"
SMB Enumeration
        │
        ▼
Usernames Identified
        │
        ▼
Hydra Dictionary Attack
        │
        ▼
SSH Login as jan
        │
        ▼
Linux Enumeration
        │
        ▼
SSH Private Key Found
        │
        ▼
ssh2john Conversion
        │
        ▼
John the Ripper
        │
        ▼
SSH Login as kay
```

This illustrates how information gathered during earlier phases enabled privilege escalation without exploiting a software vulnerability.

---

# 58. Security Findings Identified During This Phase

| Finding                            | Severity |
| ---------------------------------- | -------- |
| Exposed SSH Private Key            | Critical |
| Weak SSH Key Passphrase            | High     |
| Authentication Material Accessible | High     |
| Offline Password Recovery Possible | High     |

---

## Risk Analysis

### Confidentiality

Authentication credentials could be recovered.

### Integrity

Attackers could impersonate another user.

### Availability

Unauthorized users could gain persistent SSH access.

---

# 59. Why Offline Cracking Was Effective

## Technical Analysis

Offline password cracking offers several advantages during authorized penetration testing:

* No rate limiting.
* No account lockout.
* No authentication logs.
* Faster password testing.

Because the encrypted key was available locally, password recovery became computational rather than network-based.

---

# 👑 Phase 7 – Post Exploitation, Privilege Escalation Validation & Final Credential Recovery

> **Target:** `10.10.43.214`
>
> **Objective:** Validate successful privilege escalation by accessing previously restricted resources, recover the final protected credential, analyze the complete attack chain, and document the security impact of the compromise.

---

# 60. Beginning the Post Exploitation Phase

## 60.1 Objective

After successfully authenticating as the user **kay**, the penetration testing engagement entered the **post exploitation** stage.

The objective of this phase was to validate that privilege escalation had succeeded by attempting to access files that were inaccessible while logged in as `jan`.

This phase focuses on verifying the practical impact of privilege escalation rather than discovering new vulnerabilities.

### Objectives of This Phase

* Verify access to restricted resources.
* Access protected backup credentials.
* Validate privilege escalation success.
* Analyze the complete compromise path.
* Document the security implications.

---

## 60.2 Why `pass.bak` Became the Primary Target

During Phase 4, enumeration revealed a file named **`pass.bak`** inside `/home/kay`.

When accessed as `jan`, Linux returned a **Permission Denied** error because the file belonged to another user.

That result suggested:

* The file contained sensitive information.
* Only the owner (`kay`) could access it.
* Successful privilege escalation should grant access.

Therefore, `pass.bak` became the validation target after authenticating as `kay`.

---

# 61. Confirming Current User Context

## 61.1 Verify Active User

### Command Used

```bash
whoami
```

### Observation

The command returned:

```text
kay
```

### Analysis

This confirmed that the current shell session was operating under the `kay` account and inherited the permissions assigned to that user.

This verification is an important penetration testing practice before attempting post-exploitation actions.

---

### 📷 Screenshot Placeholder

**Figure 61.1 — Verifying Current User as `kay`**

> Insert screenshot showing the output of `whoami`.

---

## 61.2 Confirm User Identity and Groups

### Command Used

```bash
id
```

### Purpose

Display:

* User ID.
* Group memberships.
* Primary group.
* Effective identity.

### Observation

The output showed the UID and GID associated with `kay`.

### Analysis

The session was authenticated through SSH key authentication rather than the original `jan` credentials.

---

### 📷 Screenshot Placeholder

**Figure 61.2 — Identity Information of User `kay`**

> Insert screenshot showing the output of `id`.

---

# 62. Accessing the Previously Restricted File

## 62.1 Reading `pass.bak`

### Command Used

```bash
cat pass.bak
```

### Objective

Attempt to read the backup password file that previously generated a permission error.

### Observation

The command executed successfully.

The file contents became readable under the `kay` account.

---

### 📷 Screenshot Placeholder

**Figure 62.1 — Successfully Reading `pass.bak`**

> Insert screenshot showing successful execution of `cat pass.bak`.

---

## 62.2 Comparison with Previous Attempt

| Attempt | User  | Result            |
| ------- | ----- | ----------------- |
| Phase 4 | `jan` | Permission Denied |
| Phase 7 | `kay` | File Accessible   |

### Analysis

This comparison demonstrates successful privilege escalation.

The difference in access resulted entirely from user permissions rather than a change to the file itself.

---

## 62.3 Security Significance

This validates one of the most important Linux security principles:

**File ownership determines resource accessibility.**

The penetration testing objective was achieved because the assessment successfully transitioned from a lower-privileged account to an account authorized to access sensitive information.

---

# 63. Analysis of `pass.bak`

## 63.1 What Was Stored

The file contained a protected credential stored as a backup.

### Security Observation

Sensitive authentication information was stored in a user-accessible backup file.

### Why This Is Risky

Backup files often contain:

* Passwords.
* Configuration secrets.
* API keys.
* Recovery credentials.

These files are frequently forgotten during security reviews.

---

### 📷 Screenshot Placeholder

**Figure 63.1 — Contents of `pass.bak`**

> Insert screenshot of the file contents. If publishing publicly, redact the credential value.

---

## 63.2 Why This Was a Security Vulnerability

Although the file permissions prevented `jan` from reading it, multiple earlier vulnerabilities allowed access.

The attacker was able to become the file owner through privilege escalation.

### Root Cause

* Exposed authentication material.
* Weak passphrase.
* Weak password policy.
* Poor credential storage practices.

---

# 64. Validation of Privilege Escalation

## 64.1 What Changed After Escalation?

| Capability             | `jan`   | `kay` |
| ---------------------- | ------- | ----- |
| SSH Access             | ✅       | ✅     |
| Read `pass.bak`        | ❌       | ✅     |
| Access `kay` Resources | Limited | Full  |
| Read SSH Files         | Limited | Full  |

---

## 64.2 Why This Matters

Privilege escalation does not always mean obtaining root access.

In this assessment, moving from `jan` to `kay` demonstrated:

* Increased privileges.
* Access to restricted files.
* Access to sensitive authentication material.

This satisfies the privilege escalation objective defined by the TryHackMe room.

---

# 65. Complete Attack Path Analysis

## 65.1 Timeline of the Engagement

| Phase             | Outcome                          |
| ----------------- | -------------------------------- |
| VPN Connection    | Connected to lab environment.    |
| Nmap Enumeration  | Identified SSH, HTTP, SMB.       |
| Web Enumeration   | Found `/development`.            |
| File Enumeration  | Discovered developer notes.      |
| SMB Enumeration   | Retrieved usernames.             |
| Hydra             | Obtained SSH credentials.        |
| SSH Login         | Accessed target as `jan`.        |
| Linux Enumeration | Found `kay` and SSH key.         |
| SSH Key Cracking  | Recovered passphrase.            |
| SSH Key Login     | Authenticated as `kay`.          |
| Post Exploitation | Accessed restricted backup file. |

---

## 65.2 Visual Attack Chain

```text
Internet (TryHackMe VPN)
          │
          ▼
   Network Reconnaissance
          │
          ▼
     Apache Web Server
          │
          ▼
  Hidden Development Files
          │
          ▼
 SMB Enumeration (Anonymous Share)
          │
          ▼
 Valid Usernames Discovered
          │
          ▼
 Hydra Password Assessment
          │
          ▼
 SSH Login as jan
          │
          ▼
 Local Linux Enumeration
          │
          ▼
 SSH Private Key Discovery
          │
          ▼
 Offline Passphrase Cracking
          │
          ▼
 SSH Login as kay
          │
          ▼
 Access to Restricted Credentials
```

---

# 66. Security Impact Analysis

## 66.1 Confidentiality Impact

Sensitive information became accessible through chained vulnerabilities.

Examples include:

* Usernames.
* Development information.
* SSH authentication material.
* Backup credentials.

### Impact Rating

**High**

---

## 66.2 Integrity Impact

The attacker successfully authenticated as another legitimate user.

Potential consequences include:

* Unauthorized account access.
* Modification of user-owned files.
* Persistent SSH authentication.

### Impact Rating

**High**

---

## 66.3 Availability Impact

Although no denial-of-service attack occurred, unauthorized authenticated access could affect service availability if malicious actions were performed.

### Impact Rating

**Medium**

---

# 67. Root Cause Analysis

The successful compromise resulted from multiple independent weaknesses rather than a single vulnerability.

## Root Cause Breakdown

| Weakness                  | Security Impact                  |
| ------------------------- | -------------------------------- |
| Anonymous SMB Share       | Username disclosure.             |
| Exposed Development Files | Information disclosure.          |
| Weak Password Policy      | Initial SSH access.              |
| Exposed SSH Private Key   | Authentication material leakage. |
| Weak SSH Passphrase       | Offline credential recovery.     |
| Sensitive Backup File     | Protected credential disclosure. |

---

## 67.1 Vulnerability Chaining

No single vulnerability fully compromised the machine.

Instead, each weakness enabled the next phase.

```text
Information Disclosure
        │
        ▼
Username Discovery
        │
        ▼
Weak Authentication
        │
        ▼
Initial Access
        │
        ▼
Credential Enumeration
        │
        ▼
Privilege Escalation
        │
        ▼
Sensitive Data Access
```

### Security Observation

This demonstrates the importance of defense-in-depth.

Even medium-severity vulnerabilities become dangerous when combined.

---

# 68. Evidence Collected During the Assessment

## Enumeration Evidence

* Open ports identified.
* Apache server identified.
* Samba service identified.

### Screenshot Reference

Figure 3.1

---

## Information Disclosure Evidence

* `/development`
* `dev.txt`
* `j.txt`

### Screenshot Reference

Figures 8.2, 10.1, 11.1

---

## SMB Evidence

* Anonymous share.
* `staff.txt`
* Usernames.

### Screenshot Reference

Figures 16.1–19.1

---

## Authentication Evidence

* Hydra success.
* SSH login.

### Screenshot Reference

Figures 26.1, 27.1

---

## Privilege Escalation Evidence

* `id_rsa`
* John cracking.
* SSH login as `kay`.

### Screenshot Reference

Figures 42.1–55.2

---

## Final Evidence

* `pass.bak`
* Successful credential retrieval.

### Screenshot Reference

Figures 62.1 and 63.1

---

# 69. Why Root Access Was Not Pursued

## Assessment Objective

The TryHackMe Basic Pentesting room defines privilege escalation through successful access to the intended user account and protected resources.

### Ethical Reasoning

The objective was to demonstrate:

* Enumeration.
* Authentication.
* Privilege escalation.
* Security reporting.

Attempting unnecessary persistence or destructive actions was outside the scope of the internship.

---

# 70. Lessons Learned During Post Exploitation

## Technical Lessons

* Linux permissions determine access to sensitive resources.
* SSH keys require secure storage and permissions.
* Backup credential files increase attack surface.
* Offline password cracking is effective when authentication material is exposed.

---

## Penetration Testing Lessons

* Always verify privilege changes using `whoami` and `id`.
* Preserve evidence instead of modifying target files.
* Document each successful access with screenshots.
* Correlate findings across multiple attack surfaces.

---

# 📑 Phase 8 – Security Assessment Report, Risk Analysis & Internship Conclusion

> **Target:** `10.10.43.214`
>
> **Objective:** Summarize the complete Vulnerability Assessment and Penetration Testing engagement, document identified vulnerabilities, assess business impact, provide remediation recommendations, and conclude the ShadowFox Cyber Security Internship project.

---

# 72. Executive Summary

## 72.1 Assessment Overview

As part of the **ShadowFox Cyber Security Internship**, a complete Vulnerability Assessment and Penetration Testing (VAPT) engagement was performed against the **TryHackMe Basic Pentesting** laboratory machine (`10.10.43.214`).

The assessment simulated a real-world internal penetration testing engagement within an authorized and isolated cybersecurity training environment.

The engagement followed a structured penetration testing methodology including:

* Reconnaissance
* Service Enumeration
* Web Enumeration
* SMB Enumeration
* Authentication Assessment
* Linux Enumeration
* Privilege Escalation
* Post Exploitation
* Security Reporting

The objective was not only to identify vulnerabilities but also to demonstrate how multiple weaknesses could be chained together to gain unauthorized access to protected resources.

---

## 72.2 Assessment Scope

| Assessment Parameter   | Details                                                 |
| ---------------------- | ------------------------------------------------------- |
| Assessment Type        | Internal Vulnerability Assessment & Penetration Testing |
| Environment            | TryHackMe Controlled Lab                                |
| Target Host            | `10.10.43.214`                                          |
| Target Platform        | Linux                                                   |
| Web Server             | Apache HTTP Server                                      |
| SMB Service            | Samba                                                   |
| Authentication Service | OpenSSH                                                 |

---

## 72.3 Overall Assessment Outcome

The assessment successfully achieved all defined objectives.

### Objectives Completed

* VPN connectivity established.
* Target machine enumerated.
* Hidden web resources discovered.
* SMB shares enumerated.
* Valid usernames identified.
* SSH authentication weakness identified.
* Initial shell access obtained.
* Linux privilege escalation performed.
* Restricted credential successfully accessed.
* Vulnerabilities documented with remediation recommendations.

---

# 73. Vulnerability Assessment Summary

## 73.1 Identified Vulnerabilities

| ID   | Vulnerability                        | Severity    |
| ---- | ------------------------------------ | ----------- |
| V-01 | Anonymous SMB Share Enabled          | 🔴 High     |
| V-02 | Public Development Directory Exposed | 🟡 Medium   |
| V-03 | Username Enumeration Through SMB     | 🔴 High     |
| V-04 | Weak SSH Password Policy             | 🔴 High     |
| V-05 | Exposed SSH Private Key              | 🔴 Critical |
| V-06 | Weak SSH Private Key Passphrase      | 🔴 High     |
| V-07 | Sensitive Backup Password File       | 🔴 High     |

---

## 73.2 Severity Distribution

| Severity | Number of Findings |
| -------- | ------------------ |
| Critical | 1                  |
| High     | 5                  |
| Medium   | 1                  |
| Low      | 0                  |

### Overall Risk Rating

**High Risk**

The target machine contained multiple authentication and information disclosure weaknesses that could be combined into a successful compromise.

---

# 74. Detailed Risk Analysis

## 74.1 Confidentiality Impact

The assessment demonstrated several confidentiality risks.

### Information Exposed

* Developer notes.
* Internal usernames.
* SSH authentication material.
* Backup credentials.

### Business Impact

If this configuration existed in a production environment, attackers could gain unauthorized access to confidential organizational information.

**Impact Rating:** High

---

## 74.2 Integrity Impact

The attacker successfully authenticated as a second legitimate user.

Potential consequences include:

* Modifying user-owned files.
* Accessing restricted resources.
* Uploading unauthorized SSH keys.

**Impact Rating:** High

---

## 74.3 Availability Impact

Although no denial-of-service techniques were used, unauthorized authenticated access increases the possibility of service disruption.

Potential risks include:

* File deletion.
* Service modification.
* Persistence mechanisms.

**Impact Rating:** Medium

---

## 74.4 CIA Triad Summary

| Security Principle | Impact |
| ------------------ | ------ |
| Confidentiality    | High   |
| Integrity          | High   |
| Availability       | Medium |

---

# 75. Attack Surface Assessment

The penetration testing assessment identified three primary attack surfaces.

## 75.1 HTTP Service

### Exposure

* Apache web server.
* Hidden development directory.
* Directory listing enabled.

### Risk

Information disclosure.

### Intelligence Obtained

* Development notes.
* User hints.
* Technology references.

---

## 75.2 SMB Service

### Exposure

* Anonymous share.
* User enumeration.
* Public staff information.

### Risk

Username disclosure and sensitive file exposure.

---

## 75.3 SSH Service

### Exposure

* Password authentication enabled.
* Weak password accepted.
* SSH key authentication available.

### Risk

Unauthorized remote shell access.

---

# 76. Vulnerability Chaining Analysis

One of the most important lessons from this assessment is that no single vulnerability caused complete compromise.

Instead, vulnerabilities were chained together.

## Attack Chain

```text id="e1x7mr"
Information Disclosure
        │
        ▼
Username Discovery
        │
        ▼
Weak Password Authentication
        │
        ▼
Initial Shell Access
        │
        ▼
Linux Enumeration
        │
        ▼
Private SSH Key Discovery
        │
        ▼
Offline Passphrase Recovery
        │
        ▼
Privilege Escalation
        │
        ▼
Sensitive Credential Disclosure
```

### Analysis

This demonstrates how attackers combine multiple weaknesses to escalate privileges without exploiting software vulnerabilities.

---

# 77. Security Recommendations

## 77.1 Disable Anonymous SMB Access

### Issue

Anonymous users accessed shared resources without authentication.

### Recommendation

* Disable guest SMB access.
* Require authenticated SMB sessions.
* Restrict shares to authorized users only.

### Benefit

Prevents unauthorized access to internal files.

---

## 77.2 Disable Directory Listing

### Issue

Apache exposed the `/development` directory.

### Recommendation

Disable directory indexing inside Apache configuration.

Example:

```apache id="mqdwbt"
Options -Indexes
```

### Benefit

Prevents browsing hidden directories.

---

## 77.3 Remove Development Files from Production

### Issue

Developer notes were accessible publicly.

### Recommendation

* Remove testing files before deployment.
* Store development documentation outside the web root.
* Restrict development environments.

### Benefit

Reduces information disclosure during reconnaissance.

---

## 77.4 Strengthen Password Policy

### Issue

SSH password vulnerable to dictionary attack.

### Recommendation

* Minimum password length.
* Complexity requirements.
* Password expiration policy.
* Failed login lockout.

### Benefit

Mitigates brute-force and dictionary attacks.

---

## 77.5 Harden SSH Configuration

### Issue

Password authentication enabled.

### Recommendation

* Disable password authentication where possible.
* Allow SSH key authentication only.
* Disable root login.
* Restrict login attempts.

Example SSH Configuration

```text id="mjlwmn"
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
```

### Benefit

Reduces authentication attack surface.

---

## 77.6 Secure SSH Private Keys

### Issue

Private key exposure enabled offline attacks.

### Recommendation

* Store keys only inside user-owned directories.
* Apply permissions:

```bash id="hklhmx"
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
```

### Benefit

Protects authentication material.

---

## 77.7 Use Strong SSH Passphrases

### Issue

Weak passphrase cracked successfully.

### Recommendation

* Use randomly generated passphrases.
* Avoid dictionary words.
* Use password managers.

### Benefit

Improves resistance to offline cracking.

---

## 77.8 Protect Sensitive Backup Files

### Issue

Credentials stored inside `pass.bak`.

### Recommendation

* Avoid plaintext credentials.
* Encrypt backup files.
* Use secrets management solutions.
* Restrict file ownership.

### Benefit

Reduces credential exposure risk.

---

# 78. Security Best Practices Learned

The assessment reinforced several Linux and network security best practices.

## Authentication Best Practices

* Enforce Multi-Factor Authentication.
* Disable unused accounts.
* Rotate SSH keys periodically.
* Audit failed login attempts.

---

## Linux Security Best Practices

* Principle of Least Privilege.
* Secure `.ssh` directory permissions.
* Remove unnecessary files.
* Restrict home directory permissions.

---

## Web Security Best Practices

* Disable directory indexing.
* Remove development artifacts.
* Sanitize deployment environments.
* Review exposed files before publishing.

---

## SMB Security Best Practices

* Disable guest sessions.
* Disable unnecessary shares.
* Restrict SMB enumeration.
* Audit Samba configuration regularly.

---

# 79. Tools Used During the Internship

| Tool                 | Purpose                                 |
| -------------------- | --------------------------------------- |
| Kali Linux           | Penetration testing operating system.   |
| OpenVPN              | Secure connectivity to lab environment. |
| Nmap                 | Reconnaissance and service enumeration. |
| HTTP Enumeration NSE | Hidden directory discovery.             |
| Enum4Linux           | SMB enumeration.                        |
| SMBClient            | Anonymous share access.                 |
| Hydra                | SSH password assessment.                |
| SSH                  | Remote shell access.                    |
| ssh2john.py          | SSH key conversion.                     |
| John the Ripper      | Offline password cracking.              |

---

# 80. Skills Gained During the Internship

## Offensive Security Skills

* Network Reconnaissance.
* Service Enumeration.
* SMB Enumeration.
* Password Auditing.
* Linux Enumeration.
* Privilege Escalation.
* SSH Authentication Analysis.

---

## Linux Skills

* User enumeration.
* File permission analysis.
* Hidden file inspection.
* SSH configuration analysis.
* Command-line navigation.

---

## Cybersecurity Reporting Skills

* Documenting findings.
* Severity classification.
* Root cause analysis.
* Mitigation recommendations.
* Evidence documentation.

---

# 81. Challenges Faced During the Assessment

## Challenge 1 — Identifying Valid Usernames

### Problem

Initially only user initials were available.

### Solution

SMB enumeration revealed complete usernames through anonymous shares.

---

## Challenge 2 — Accessing Restricted Files

### Problem

`pass.bak` generated a permission error.

### Solution

Performed privilege escalation through SSH key authentication.

---

## Challenge 3 — Understanding SSH Key Authentication

### Problem

Private key was encrypted.

### Solution

Converted the key using `ssh2john.py` and recovered its passphrase offline.

---

## Challenge 4 — Linux Permission Analysis

### Problem

Determining which resources were accessible.

### Solution

Used Linux enumeration commands to inspect ownership and permissions before attempting access.

---

# 82. Learning Outcomes

This internship significantly improved both theoretical understanding and practical penetration testing skills.

## Technical Learning Outcomes

* Understanding penetration testing workflow.
* Identifying attack surfaces.
* Enumerating Linux services.
* SMB security testing.
* Password auditing methodology.
* SSH authentication mechanisms.
* Offline credential attacks.
* Linux privilege escalation.

---

## Professional Learning Outcomes

* Structured vulnerability reporting.
* Evidence collection.
* Security documentation.
* Risk prioritization.
* Ethical penetration testing methodology.

---

# 83. Mapping Internship Activities to VAPT Lifecycle

| VAPT Stage              | Activity Performed                             |
| ----------------------- | ---------------------------------------------- |
| Reconnaissance          | Nmap scanning.                                 |
| Enumeration             | HTTP and SMB enumeration.                      |
| Vulnerability Discovery | Developer directory, SMB share, usernames.     |
| Exploitation            | SSH dictionary attack.                         |
| Initial Access          | SSH login as `jan`.                            |
| Privilege Escalation    | SSH private key authentication as `kay`.       |
| Post Exploitation       | Access restricted credential file.             |
| Reporting               | Documentation and remediation recommendations. |

---

# 84. Ethical Considerations

This assessment was conducted responsibly inside the TryHackMe platform.

### Ethical Practices Followed

* Only authorized systems were tested.
* No persistence mechanisms were created.
* No destructive commands were executed.
* No files were modified.
* The assessment remained within the internship scope.

This reflects the responsibilities expected from ethical hackers during authorized penetration testing engagements.

---

# 85. Conclusion

The **ShadowFox Cyber Security Internship** provided practical exposure to the complete Vulnerability Assessment and Penetration Testing process through the **TryHackMe Basic Pentesting** laboratory.

Beginning with reconnaissance, the assessment demonstrated how publicly exposed services and insecure configurations can reveal valuable intelligence. Systematic enumeration of HTTP and SMB services exposed developer information and valid usernames, which enabled an authorized authentication assessment against the SSH service.

After obtaining initial access as a low-privileged user, detailed Linux enumeration identified sensitive SSH authentication material. By securely converting and analyzing the encrypted SSH private key offline, privilege escalation was successfully achieved without exploiting software vulnerabilities. The assessment concluded by validating access to previously restricted resources and documenting the security impact of the identified weaknesses.

This project strengthened my practical understanding of Linux security, network enumeration, authentication mechanisms, privilege escalation, and professional VAPT reporting. It also highlighted the importance of secure configurations, strong authentication policies, proper SSH key management, and defense-in-depth in protecting Linux systems from chained attacks.

The internship successfully achieved all defined objectives and provided hands-on experience with tools and methodologies commonly used in real-world cybersecurity assessments.

---

# 86. References

The following official resources were used to understand the tools and techniques applied during this internship:

### Penetration Testing Resources

* TryHackMe – Basic Pentesting Room Documentation
* Kali Linux Official Documentation

### Tool Documentation

* Nmap Official Documentation
* Hydra Official Documentation
* Samba Documentation
* OpenSSH Documentation
* John the Ripper Documentation

### Linux References

* Linux Manual Pages (`man ssh`, `man chmod`, `man ls`)
* GNU Core Utilities Documentation

---

# Appendix A — Commands Used During the Assessment

| Phase                  | Command                                          |
| ---------------------- | ------------------------------------------------ |
| VPN Setup              | `sudo openvpn tryhackme.ovpn`                    |
| Connectivity           | `ip a`, `ping 10.10.43.214`                      |
| Network Scan           | `nmap -sV -A 10.10.43.214`                       |
| HTTP Enumeration       | `nmap --script http-enum -p80 10.10.43.214`      |
| SMB Enumeration        | `enum4linux -a 10.10.43.214`                     |
| SMB Share Access       | `smbclient //10.10.43.214/Anonymous`             |
| Password Assessment    | `hydra -l jan -P rockyou.txt ssh://10.10.43.214` |
| SSH Login              | `ssh jan@10.10.43.214`                           |
| Linux Enumeration      | `cat /etc/passwd`, `ls -la`, `whoami`, `id`      |
| SSH Key Conversion     | `python3 ssh2john.py bp > hash.txt`              |
| Password Cracking      | `john --wordlist=rockyou.txt hash.txt`           |
| SSH Key Authentication | `ssh -i bp kay@10.10.43.214`                     |
| Credential Access      | `cat pass.bak`                                   |

---

# Appendix B — Evidence Checklist

Use the following screenshots inside the documentation.

| Figure      | Screenshot Description               |
| ----------- | ------------------------------------ |
| Figure 2.1  | OpenVPN connection established.      |
| Figure 2.2  | `tun0` interface verification.       |
| Figure 2.3  | Successful ping to target.           |
| Figure 3.1  | Nmap aggressive scan output.         |
| Figure 3.2  | Open ports identified.               |
| Figure 7.1  | Target homepage.                     |
| Figure 8.1  | HTTP enumeration output.             |
| Figure 8.2  | `/development` directory discovered. |
| Figure 9.1  | Directory listing of `/development`. |
| Figure 10.1 | Contents of `dev.txt`.               |
| Figure 11.1 | Contents of `j.txt`.                 |
| Figure 15.1 | Enum4Linux enumeration output.       |
| Figure 16.1 | SMB shares identified.               |
| Figure 16.2 | User enumeration results.            |
| Figure 17.1 | Anonymous SMB login.                 |
| Figure 18.1 | SMB share contents.                  |
| Figure 18.2 | Downloading `staff.txt`.             |
| Figure 19.1 | Contents of `staff.txt`.             |
| Figure 25.1 | Hydra execution.                     |
| Figure 26.1 | Hydra successful authentication.     |
| Figure 27.1 | SSH login as `jan`.                  |
| Figure 28.1 | `whoami` output (`jan`).             |
| Figure 29.1 | `/etc/passwd` enumeration.           |
| Figure 31.1 | `ls -la /home/kay`.                  |
| Figure 39.1 | `.ssh` directory contents.           |
| Figure 42.1 | OpenSSH private key header.          |
| Figure 43.2 | `chmod 400 bp`.                      |
| Figure 49.1 | `ssh2john.py` location.              |
| Figure 50.1 | SSH key conversion.                  |
| Figure 50.2 | Generated hash file.                 |
| Figure 52.1 | John the Ripper cracking process.    |
| Figure 53.1 | Passphrase recovered.                |
| Figure 54.1 | SSH login as `kay`.                  |
| Figure 55.1 | `whoami` output (`kay`).             |
| Figure 62.1 | Successful `cat pass.bak`.           |
| Figure 63.1 | Final credential file (redacted).    |

---

## Project Status

**Assessment:** Successfully Completed

**Internship:** ShadowFox Cyber Security Internship

**Project Type:** Vulnerability Assessment & Penetration Testing (VAPT)

