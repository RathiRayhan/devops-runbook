# MySQL/MariaDB Migration & Disaster Recovery Runbook

## 📌 Overview
This runbook outlines the standard operating procedure (SOP) for safely backing up a production MySQL/MariaDB database, securely transferring it across servers using a local machine as a bastion/middleman, and accurately restoring it on the destination server.

---

## 🛠️ Phase 1: Database Backup (Export)

We use the `mysqldump` utility to generate a logical backup (`.sql` file) of the target database.

### 1. Identify the Target Database
Log into the database shell to verify the database name:
```bash
sudo mysql
MariaDB [(none)]> SHOW DATABASES;
MariaDB [(none)]> \q
```

### 2. Execute the Backup Command
To back up a database (e.g., `wordpress_db`), execute the following command:
```bash
sudo mysqldump -u root wordpress_db > wordpress_db_backup.sql
```

**💡 Authentication Context & Variations:**
*   **Unix Socket Authentication (Default on modern Linux):** By using `sudo`, we authenticate as the system `root`, allowing us to bypass the MySQL password prompt. 
*   **Password/Standard Authentication:** If Unix socket authentication is disabled, or you are using a non-root user, you MUST include the `-p` flag to prompt for a password:
    ```bash
    mysqldump -u root -p wordpress_db > wordpress_db_backup.sql
    ```

Verify the backup file was created successfully:
```bash
ls -lh wordpress_db_backup.sql
```

---

## ☁️ Phase 2: Secure Transfer (The Middleman Approach)

Transferring files directly between two remote servers (Server A to Server B) requires placing your private SSH key on Server A, which is a **massive security risk**. Instead, we use our local machine as a secure middleman.

### 1. Download to Local Machine
Pull the backup file from the source server to your local machine:
```bash
# Example with custom SSH port 5259
scp -P 5259 user@<SOURCE_IP>:/home/user/wordpress_db_backup.sql .
```

### 2. Upload to Destination Server
Push the file from your local machine to the destination server using your private key:
```bash
scp -i ~/.ssh/your_private_key wordpress_db_backup.sql admin@<DESTINATION_IP>:/home/admin/
```

---

## 🚨 Phase 3: Restoration (Import)

### ⚠️ CRITICAL WARNING: `mysqldump` vs `mysql`
A very common pitfall is attempting to restore a database using the `mysqldump` command. 
*   **`mysqldump`** is ONLY for exporting/creating backups. If you use it with a `<` (input) redirection, it will ignore the file, attempt to dump an empty database, and print the output to your terminal screen.
*   **`mysql`** is the standard client used for importing/restoring data.

### 1. Create an Empty Database
Unlike some NoSQL databases, MySQL requires the destination database to exist before importing data into it.
```bash
sudo mysql
MariaDB [(none)]> CREATE DATABASE wordpress_db;
MariaDB [(none)]> \q
```

### 2. Execute the Restore Command
Use the standard `mysql` client to import the `.sql` file into the newly created database:
```bash
sudo mysql -u root wordpress_db < wordpress_db_backup.sql
```
*(Note: A successful restoration will return no output in the terminal.)*

### 3. Verify Restoration
```bash
sudo mysql
MariaDB [(none)]> USE wordpress_db;
MariaDB [(none)]> SHOW TABLES;
```

---

## 🎩 Operational Tip: Recovering Lost Credentials

In unmanaged or legacy environments, database credentials may be undocumented or lost by the stakeholders. If you only have root SSH access, you can securely extract the active database credentials directly from the application's configuration files without needing to reset passwords and cause downtime:

*   **WordPress:** `cat /var/www/html/wp-config.php | grep DB_`
*   **Node.js/Laravel/Python:** `cat /path/to/app/.env`

These files contain the active database name, username, and password in plain text, allowing you to perform necessary administrative tasks seamlessly.
