# Topic 2: Nginx Web Server, Reverse Proxy & SSL Configuration

## Objective
Configure Nginx as a reverse proxy, host a static website, troubleshoot common permission/port errors, and secure the server with HTTPS using Let's Encrypt (Certbot).

---

## Part 1: Nginx Reverse Proxy Setup
**Use Case:** Routing traffic from standard HTTP port (80) to a backend application running on a custom port (e.g., 8080).

1. **Install Nginx:**
   ```bash
   sudo apt update
   sudo apt install nginx -y
   ```

2. **Create Configuration File:**
   ```bash
   sudo nano /etc/nginx/sites-available/simple-web-server
   ```
   *Configuration:*
   ```nginx
   server {
       listen 80;
       server_name <YOUR_VPS_IP>; # Replace with your server IP or Domain

       location / {
           proxy_pass [http://127.0.0.1:8080](http://127.0.0.1:8080);
       }
   }
   ```

3. **Enable Site via Symlink & Reload:**
   ```bash
   sudo ln -s /etc/nginx/sites-available/simple-web-server /etc/nginx/sites-enabled/
   sudo nginx -t                # Test config for syntax errors
   sudo systemctl reload nginx  # Graceful reload (Zero Downtime)
   ```

---

## Part 2: Hosting a Static Website (HTML/CSS/JS)
**Use Case:** Serving a frontend portfolio directly from the server.

1. **Transfer & Move Files:**
   Use `scp` from the local machine to transfer the code, then move it to the web root:
   ```bash
   sudo mv /path/to/transferred/files /var/www/my-portfolio
   ```

2. **Create Nginx Configuration:**
   ```bash
   sudo nano /etc/nginx/sites-available/my-portfolio
   ```
   *Configuration:*
   ```nginx
   server {
       listen 80;
       server_name <YOUR_VPS_IP>; # Will be updated to domain later

       location / {
           root /var/www/my-portfolio;
           index index.html;
       }
   }
   ```

3. **Enable and Reload:**
   ```bash
   sudo ln -s /etc/nginx/sites-available/my-portfolio /etc/nginx/sites-enabled/
   sudo systemctl reload nginx
   ```

---

## Part 3: Troubleshooting Common Nginx Errors

### Issue 1: Port 80 Conflict (Address already in use)
**Cause:** Another config (e.g., `simple-web-server`) is already listening on Port 80.
**Fix:** Remove the symlink of the conflicting site from `sites-enabled` (Do NOT delete the original file from `sites-available`).
```bash
sudo rm /etc/nginx/sites-enabled/simple-web-server
sudo systemctl reload nginx
```

### Issue 2: 403 Forbidden Error
**Cause:** Nginx worker process runs as user `www-data` and lacks read permissions for the web root directory. Checked via `sudo tail -n 20 /var/log/nginx/error.log`.
**Fix:** Correct ownership and permissions recursively.
```bash
sudo chown -R www-data:www-data /var/www/my-portfolio
sudo chmod -R 755 /var/www/my-portfolio
```

**Security Warning (.env files):** 
Ensure there are no `.env` files containing sensitive credentials inside the web root (`/var/www/...`). If present, they will be publicly exposed to the internet. Move them outside the web root or deny access via Nginx location blocks.

---

## Part 4: Domain DNS & SSL/TLS Setup (Certbot)
**Use Case:** Securing the website with a green padlock (HTTPS).

1. **DNS Setup:**
   * Log into the Domain Provider panel (e.g., Cloudflare, Namecheap).
   * Add an **A Record**: Name `@` (or `subdomain_name`), Value: `<YOUR_VPS_IP>`.

2. **Update Nginx Config:**
   Update `server_name` in `/etc/nginx/sites-available/my-portfolio` to the actual domain (e.g., `server_name <your_domain.com>;`), then reload Nginx.

3. **Firewall (UFW) Configuration:**
   Certbot requires both HTTP (80) and HTTPS (443) ports open for validation and traffic.
   ```bash
   sudo ufw allow 'Nginx Full'
   # Or explicitly: sudo ufw allow 80/tcp && sudo ufw allow 443/tcp
   sudo ufw reload
   ```

4. **Install Certbot (via Python venv - Official standard):**
   ```bash
   sudo python3 -m venv /opt/certbot/
   sudo /opt/certbot/bin/pip install --upgrade pip
   sudo /opt/certbot/bin/pip install certbot certbot-nginx
   sudo ln -s /opt/certbot/bin/certbot /usr/local/bin/certbot
   ```

5. **Generate SSL Certificate:**
   ```bash
   sudo certbot --nginx -d <your_domain.com>
   ```
   *(Follow on-screen prompts. Certbot will automatically modify the Nginx config to redirect HTTP to HTTPS).*

6. **Automate Certificate Renewal (Cronjob):**
   SSL certificates expire every 90 days. Set up a cronjob for auto-renewal.
   ```bash
   echo "0 0,12 * * * root /opt/certbot/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' && sudo certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
   ```
   **Test Auto-renewal:**
   ```bash
   sudo certbot renew --dry-run
   ```