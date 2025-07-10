# HackMyVM: Canto - Easy

![Canto Icon](Canto.png)

*   **Difficulty:** Easy 🟢
*   **Author:** DarkSpirit
*   **Date:** 23. Juni 2025
*   **VM Link:** [https://hackmyvm.eu/machines/machine.php?vm=Canto](https://hackmyvm.eu/machines/machine.php?vm=Canto)
*   **Full Report (HTML):** [Link zum vollständigen Pentest-Bericht](https://alientec1908.github.io/Canto_HackMyVM_Easy/)

## Overview

This report documents the penetration testing process of the "Canto" virtual machine from HackMyVM, rated as an Easy difficulty challenge. The objective was to identify and exploit vulnerabilities to gain root access to the system. The machine involved web application exploitation, information gathering, and privilege escalation techniques.

## Methodology

The approach followed a standard penetration testing methodology, starting with reconnaissance and enumeration to identify potential attack vectors, gaining an initial foothold, and finally escalating privileges to root.

### Reconnaissance & Web Enumeration

1.  **Host Discovery:** Identified the target IP (192.168.2.57) and hostname (`canto.hmv`).
2.  **Port Scanning (Nmap):** Discovered open ports 22 (SSH) and 80 (HTTP). Identified OpenSSH and Apache httpd.
3.  **Web Application Analysis:** Confirmed a WordPress installation on Port 80.
4.  **Directory & File Enumeration (Nikto, Gobuster):** Found standard WordPress paths (`wp-login.php`, `xmlrpc.php`, `wp-content/uploads/` with directory listing enabled), and identified missing security headers (`X-Frame-Options`, `X-Content-Type-Options`).
5.  **User Enumeration (WordPress REST API):** Successfully identified the username `erik` via the WP-JSON API endpoint (`/wp-json/wp/v2/users/1`).
6.  **WordPress Vulnerability Scanning (WPScan):** Identified the presence of the **Canto Plugin (Version 3.0.4)**, which is vulnerable to several unauthenticated issues, including **Remote File Inclusion (RFI)** and **Remote Code Execution (RCE)** (CVE-2023-3452, CVE-2024-25096). WPScan's brute-force attempt on `erik` failed using the RockYou wordlist.

### Initial Access

The critical vulnerability found in the Canto Plugin (v3.0.4) allowed for unauthenticated RCE.

1.  A publicly available Python PoC exploit for CVE-2023-3452 was located and downloaded from GitHub.
2.  A PHP reverse shell payload was configured and prepared.
3.  A Netcat listener was set up on the attacker machine.
4.  The PoC script was used to trigger the vulnerability, causing the target system to download and execute the reverse shell payload from the attacker's local HTTP server.
5.  Successfully obtained a reverse shell as the `www-data` user.

### Privilege Escalation

From the `www-data` shell, internal enumeration was performed to find a path to root.

1.  **System Enumeration:** Explored the file system, including the `/home/` directory. Discovered the user `erik`'s home directory was readable by `www-data`.
2.  **Information Gathering:** Found a readable `notes` directory within `erik`'s home. Found a `.bash_history` file in `/var/www/html/`.
3.  **Analyzing Bash History:** The history revealed steps including navigating to `/var/wordpress/backups/` and reading a file named `12052024.txt`.
4.  **Password Discovery:** Navigated to `/var/wordpress/backups/` and read `12052024.txt`, which contained the cleartext password for user `erik`: `th1sIsTheP3ssw0rd!`.
5.  **Horizontal Privilege Escalation:** Used the found password with `su erik` to switch user context to `erik`.
6.  **Sudo Rights Check:** Executed `sudo -l` as `erik` and discovered that `erik` could run `/usr/bin/cpulimit` as `ALL` users (`root`) **without a password (NOPASSWD)**.
7.  **Vertical Privilege Escalation:** Utilized the `cpulimit` Sudo permission to execute `/bin/sh` as root, using a method found on GTFOBins (`sudo cpulimit -l 100 -f /bin/sh`).
8.  Successfully obtained a root shell.

### Flags

Both the user.txt and root.txt flags were successfully retrieved after gaining the respective privileges.

---

[Link to Full Report](https://alientec1908.github.io/Canto_HackMyVM_Easy/)
