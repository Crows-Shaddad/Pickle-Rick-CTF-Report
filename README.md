# Pickle-Rick-CTF-Report
# Penetration Testing Report (Pickle Rick CTF)

---

## 1. Executive Summary
A simulated penetration testing assessment was conducted on the "Pickle Rick" target. The assessment revealed critical vulnerabilities, including Remote Code Execution (**RCE**) on the web server, which allowed initial access as the `www-data` user. Due to a severe sudo misconfiguration, privilege escalation to the **Root** user was achieved, granting full unauthorized access to all system data (the three ingredients).

---

## 2. Reconnaissance & Target Information
*   **Target Name:** Pickle Rick (TryHackMe)
*   **Active Services Discovered:**
    *   **Port 22 (SSH):** Closed or filtered.
    *   **Port 80 (HTTP):** Apache Web Server hosting a custom Rick and Morty themed page.

---

## 3. Exploitation & Privilege Escalation Chain

### Step 1: Web Enumeration & Credential Leakage
*   Inspecting the HTML source code of the main page leaked a hidden username. For security reasons, the credential has been redacted:
    `User: [REDACTED_USERNAME]`
*   Checking the `/robots.txt` file exposed a cleartext string, suspected to be a password. For security reasons, the credential has been redacted:
    `Password: [REDACTED_PASSWORD]`
*   These credentials granted successful authentication into the portal panel at `/login.php`. This panel contained a Command Execution interface.

### Step 2: Initial Access & Command Restriction Bypass
*   Direct execution of the `cat` command was blocked by a server-side blacklist.
*   **Bypass Technique:** Alternative text-viewing utilities such as `less`, `grep .`, or `strings` were used.
*   **First Ingredient:** Located within the web root directory.
    *   **Path:** `/var/www/html/Sup3rS3cretPickl3Ingred.txt`
    *   **Content:** `mr. meeseek hair`

### Step 3: Lateral Movement (File System Enumeration)
*   The system's home directories were enumerated using `ls /home`.
*   A home directory for the user `rick` was identified.
*   **Second Ingredient:** Located inside Rick's home directory.
    *   **Path:** `/home/rick/second ingredients`
    *   **Content:** `1 jerry tear`

### Step 4: Privilege Escalation to Root
*   The privileges of the current low-privileged user (`www-data`) were evaluated via:
    ```bash
    sudo -l
    ```
*   **Misconfiguration Found:** The user was permitted to run all commands as root without a password:
    `(ALL : ALL) NOPASSWD: ALL`
*   **Third Ingredient:** Acquired by leveraging the unrestricted sudo privilege to read the root flag.
    *   **Exploit Command:** `sudo less /root/3rd.txt`
    *   **Content:** `fleeb juice`

---

## 4. Vulnerabilities & Remediation Matrix

| Discovered Vulnerability | Severity | Remediation Strategy |
| :--- | :---: | :--- |
| **Remote Code Execution (RCE)** | Critical | Completely remove system command execution functions from the web application. Implement strict input sanitization. |
| **Exposed Credentials in Source/Robots** | High | Remove sensitive information, comments, and passwords from public-facing directories and source code. |
| **Sudo Misconfiguration (Sudo -l)** | Critical | Enforce the Principle of Least Privilege. Never assign `NOPASSWD: ALL` permissions to a web server service account. |
