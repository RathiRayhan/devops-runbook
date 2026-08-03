# Topic 1: Linux Server Setup, SSH Hardening & Firewall (UFW)

**Objective:** Set up a secure baseline for a Linux server (Ubuntu/Debian) suitable for production environments. This runbook covers user management, key-based authentication, SSH service hardening, and network security using UFW.

---

## 1. User Management and Permissions
**Concept:** Never operate as the `root` user to avoid catastrophic accidental system changes. Create a standard user and grant administrative (`sudo`) privileges.

*   **Login as root:** `ssh root@[server_ip]`
*   **Create a new user:** `adduser [username]`
*   **Grant Sudo Access (Method 1 - Standard):** `usermod -aG sudo [username]`
*   **Grant Sudo Access (Method 2 - Manual `visudo`):**
    *   Command: `EDITOR=nano sudo visudo` (Forces nano editor if default is not set).
    *   Syntax logic in `sudoers` file: `[Username] [Host]=(RunAsUser:RunAsGroup) [Commands]`
    *   Example: `harry ALL=(ALL:ALL) ALL`
*   **Switch User:** `su - [username]`
*   **File Permissions Basics:** Learned how to change owners (`chown`) and modify file permissions (`chmod`).

---

## 2. Secure Shell (SSH) & Key-Based Authentication
**Concept:** Password logins are vulnerable to brute-force attacks. Key-pair authentication is the industry standard.

*   **Generate SSH Key Pair (On Host Machine):** 
    `ssh-keygen -t ed25519` (Uses the fastest and most secure algorithm).
*   **Deploy Public Key:** The public key must be placed in the remote server's `~/.ssh/authorized_keys` file. (Can be done manually or via `ssh-copy-id [user]@[ip]`).
*   **Strict Permission Requirements (Crucial):** If permissions are too open, the SSH daemon will reject the key and fallback to asking for a password.
    *   `chmod 700 ~/.ssh` (Only the owner can read/write/execute the directory).
    *   `chmod 600 ~/.ssh/authorized_keys` (Only the owner can read/write the file).
*   **Service Logging:** To monitor SSH login attempts or troubleshoot failures:
    `sudo journalctl -u ssh.service` (Note: Ubuntu uses `ssh.service`, while Arch/CentOS uses `sshd.service`).

---

## 3. SSH Hardening (Securing the Gates)
**Concept:** Prevent automated bot attacks by changing the default port and explicitly denying root and password-based logins.

*   **Edit Main Config:** `sudo nano /etc/ssh/sshd_config`
*   **Modifications:**
    1.  Change default port: `Port 22` -> `Port 5259`
    2.  Disable root login: `PermitRootLogin no`
    3.  Disable passwords: `PasswordAuthentication no`
*   **Verify Syntax:** `sudo sshd -t` (No output means the configuration is error-free).
*   **The `.d` Directory Override Issue (Gotcha):**
    *   Configurations inside `/etc/ssh/sshd_config.d/*.conf` override the main `sshd_config` file. If `PasswordAuthentication no` is ignored, check this directory and comment out conflicting lines.
*   **Apply Changes (Socket System):** On modern Ubuntu, SSH uses socket activation.
    *   `sudo systemctl daemon-reload`
    *   `sudo systemctl restart ssh.socket`
    *   `sudo systemctl restart ssh`
*   **Testing Hardening:** 
    *   Test password fallback: `ssh -o PubKeyAuthentication=no -p 5259 [username]@[server_ip]` (This MUST fail if hardening is successful).

---

## 4. Uncomplicated Firewall (UFW)
**Concept:** Follow the principle of least privilege. Deny all incoming traffic by default, and only open ports that are strictly necessary for your services.

*   **Install & Check Status:** 
    `sudo apt install ufw`
    `sudo ufw status verbose`
*   **Set Default Policies:**
    `sudo ufw default deny incoming`
    `sudo ufw default allow outgoing`
*   **Open Required Ports (CRITICAL SEQUENCE):** 
    *   *WARNING: Always allow your custom SSH port before enabling the firewall to prevent locking yourself out.*
    *   `sudo ufw allow 5259/tcp` (Custom SSH Port)
    *   `sudo ufw allow 80/tcp` (HTTP - Web Traffic)
    *   `sudo ufw allow 443/tcp` (HTTPS - Secure Web Traffic)
    *   *Note:* Appending `/tcp` is best practice. If omitted, UFW opens both TCP and UDP. We only need TCP for these services.
*   **Enable Firewall:** `sudo ufw enable`
*   **ICMP (Ping) Masking:** 
    *   To prevent external scanners from verifying the server is online via ping, modify the rules:
    *   `sudo nano /etc/ufw/before.rules`
    *   Find the `ok icmp codes` section and change the `echo-request` rule from `ACCEPT` to `DROP`.
*   **Deny vs. Reject:** 
    *   `REJECT`: Blocks the connection but politely informs the sender.
    *   `DENY` (`DROP`): Silently drops the packet (Preferred for security to waste attackers' time).
