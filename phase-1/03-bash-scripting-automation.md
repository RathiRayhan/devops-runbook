1. Bash Scripting


## 📌 Overview
This script automates standard server provisioning tasks: verifying/creating specific application directories and bulk-creating system users silently without interactive prompts.

## ⚙️ Prerequisites
- **OS:** Ubuntu/Debian (Linux)
- **Permissions:** `sudo` access required
- **Shell:** Bash (`/usr/bin/bash`)

## 🚀 The Script (`setup.sh`)

```bash
#!/usr/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e 

DIR_PATH="/opt/myapp-data2"

# 1. Directory Provisioning
if [ -d "$DIR_PATH" ]; then
        echo "[INFO] Folder '$DIR_PATH' already exists."
else
        echo "[INFO] Folder does not exist. Creating..."
        sudo mkdir -p "$DIR_PATH"
        echo "[SUCCESS] Folder '$DIR_PATH' created successfully."
fi

# 2. User Provisioning
for i in {1..3}; do
        # -m flag creates the user's home directory
        sudo useradd -m "another-test$i"
        echo "[SUCCESS] User 'another-test$i' created."
done
```

## 🛠️ Troubleshooting & Key Learnings
- **The `set -e` Fail-Safe:** Highly recommended for production scripts. It prevents the script from executing subsequent commands if a prior command (e.g., `mkdir` failing due to a lack of `sudo`) fails.
- **Loop Syntax (`{ }` vs `[ ]`):** In Bash, ranges in `for` loops must use curly braces (e.g., `{1..3}`). Using square brackets `[1..3]` is treated as literal characters and will result in invalid syntax or malformed usernames.
- **`useradd` vs `adduser`:**
    - `adduser`: A high-level, interactive script. Unsuitable for silent automation as it stops execution to prompt for passwords and user details.
    - `useradd`: A low-level binary. Perfect for non-interactive scripts. Always append the `-m` flag to ensure the user's home directory is created.
    
    

2. Cronjob

## Day 2: Cron Job Automation & Production Logging

### 1. Cron Fundamentals & Syntax
Cron is a time-based job scheduler in Linux used to automate repetitive tasks (e.g., health checks, backups, log rotations).

The crontab syntax consists of 5 time-and-date fields followed by the command:
```bash
* * * * * command_to_execute
┬ ┬ ┬ ┬ ┬
│ │ │ │ │
│ │ │ │ └─ Day of week (0 - 7) (0 or 7 is Sunday)
│ │ │ └─── Month (1 - 12)
│ │ └───── Day of month (1 - 31)
│ └─────── Hour (0 - 23)
└───────── Minute (0 - 59)
```

* **Wildcards & Operators:**
  * `*` : Every/All.
  * `*/n` : Step values (e.g., `*/2` in the first field means every 2 minutes).
  * `@reboot` : Runs once at system startup.

### 2. Production Best Practices for Cron
Cron executes in a highly restricted environment with a minimal `$PATH`. To prevent "Command not found" errors, strictly adhere to the following industry standards:

* **Script Location:** Never store production scripts in the user's home directory (`~/`). 
  * System-wide scripts: `/opt/scripts/` or `/usr/local/bin/`
* **Absolute Paths:** Always use the full absolute path for both the script and the internal commands within the script. Find command paths using `which <command>`.
* **Environment Variables:** Explicitly define the `PATH` at the top of the crontab file.
* **Disable Default Mail:** Prevent cron from filling up the local mail spool by disabling email alerts. Add `MAILTO=""` at the top of the crontab.

### 3. Logging & Error Handling (`2>&1`)
Since cron runs in the background without a terminal, standard output (stdout) and standard error (stderr) must be explicitly captured.

```bash
>> /var/log/health.log 2>&1
```
**Breakdown:**
* `>> /var/log/health.log` : Appends the standard output (Channel 1 - success logs) to the specified file.
* `2>&1` : Redirects Channel 2 (Standard Error) to the same location as Channel 1.
* **Result:** Both success messages and error traces are securely logged in a single file for easy troubleshooting.

### 4. Implementation: Basic vs. Production Standard
Open the cron editor: `crontab -e`

❌ **Basic/Learning Setup (Prone to failure):**
```bash
*/2 * * * * ./health_check.sh >> /tmp/health_log.txt
```
*(Fails because `./` relies on current working directory, lacks error redirection, and uses temporary storage.)*

✅ **Standard Setup:**
```bash
MAILTO=""
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Run system health check every 2 minutes and capture all logs
*/2 * * * * /opt/scripts/health_check.sh >> /var/log/health_check.log 2>&1
```

### 5. Managing Cronjobs
* **List active jobs:** `crontab -l`
* **Safely disable a job:** Never delete the line. Comment it out using `#` at the beginning of the line.
```bash
# */2 * * * * /opt/scripts/health_check.sh >> /var/log/health_check.log 2>&1
```

### 6. Disaster Recovery & Troubleshooting Simulation
**Scenario:** A cronjob is scheduled but the expected outcome isn't happening.

1. **Check if Cron fired the job:**
   ```bash
   sudo journalctl -u cron --since "1 hour ago"
   ```
   *Note: If `journalctl` shows the command was executed successfully by the cron daemon, the issue is inside the script itself, not cron.*
2. **Check the custom log file for script errors:**
   ```bash
   cat /var/log/health_check.log
   ```
   *Simulation Result:* Found `Permission denied` error.
3. **Root Cause Fix:** The script lacked execution permissions. Fixed via:
   ```bash
   sudo chmod +x /opt/scripts/health_check.sh
   ```
