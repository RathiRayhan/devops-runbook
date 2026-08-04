# Nginx Runbook: Reverse Proxy & Static Web Hosting

## 1. Reverse Proxy Setup (For Running Applications)
**Objective:** Route external web traffic from port 80 to an internal application running on a specific port (e.g., 8080).

### Steps:
1. **Create Configuration File:**
   ```bash
   sudo nano /etc/nginx/sites-available/simple-web-server
   ```
2. **Apply Configuration:**
   ```nginx
   server {
       listen 80;
       server_name 23.166.40.61; # Replace with actual IP or Domain

       location / {
           proxy_pass [http://127.0.0.1:8080](http://127.0.0.1:8080);
       }
   }
   ```
3. **Enable Site via Symlink:**
   ```bash
   sudo ln -s /etc/nginx/sites-available/simple-web-server /etc/nginx/sites-enabled/
   ```
4. **Test and Reload:**
   *Note: Using `reload` is the industry standard for zero-downtime configuration updates. Using `restart` will drop active client connections.*
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

## 2. Static Web Hosting (HTML/CSS/JS/React Build)
**Objective:** Host pre-built static files directly using Nginx without any backend application running.

### Steps:
1. **Transfer and Move Files:**
   Copy files from the local machine (via `scp`) to the server, then move them to the standard web directory.
   ```bash
   sudo mkdir -p /var/www/my-portfolio
   sudo mv ~/uploaded_build_files/* /var/www/my-portfolio/
   ```
2. **Create Configuration File:**
   ```bash
   sudo nano /etc/nginx/sites-available/my-portfolio
   ```
3. **Apply Configuration:**
   ```nginx
   server {
       listen 80;
       server_name 23.166.40.61;

       location / {
           root /var/www/my-portfolio;
           index index.html;
       }
   }
   ```
4. **Enable and Restart:**
   ```bash
   sudo ln -s /etc/nginx/sites-available/my-portfolio /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```

## 3. Troubleshooting & Security Best Practices
*   **Port Conflict / Conflicting Server Name:** If a new configuration conflicts with an old one on the same port/server_name, delete the old symlink and reload.
    ```bash
    sudo rm /etc/nginx/sites-enabled/simple-web-server
    sudo systemctl reload nginx
    ```
*   **403 Forbidden Error:** This is typically a permission denial. Nginx runs as the `www-data` user and needs specific ownership and execution rights to read the files.
    *   *Check Logs:* `sudo tail -n 20 /var/log/nginx/error.log`
    *   *Fix Permissions:*
        ```bash
        sudo chown -R www-data:www-data /var/www/my-portfolio
        sudo chmod -R 755 /var/www/my-portfolio
        ```
*   **Security Warning (Dotfiles Leakage):** Never leave sensitive files (e.g., `.env`, `.git`, `.github`) in the public web root directory. Nginx will serve them to anyone who requests the exact URI, leading to credential leaks.
    *   *Action:* Always delete or exclude them from `/var/www/` during deployment.
    ```bash
    sudo rm -rf /var/www/my-portfolio/.env /var/www/my-portfolio/.git
    ```
