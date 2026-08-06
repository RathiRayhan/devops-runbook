# Runbook: Basic Server Automation (Directory & User Provisioning)

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
