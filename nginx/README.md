# **NGINX - Comprehensive Guide**

## **1. Introduction to NGINX**

NGINX is a high-performance web server, reverse proxy, and load balancer. It is known for its scalability, security, and efficiency in handling concurrent connections.

---

## **2. Basic Linux Commands (Prerequisite Knowledge)**

Before working with NGINX, here are some essential Linux commands:

```bash
# Update package lists
sudo apt update

# Install a package
sudo apt install <package_name>

# Start, Stop, Restart a service
sudo systemctl start <service>
sudo systemctl stop <service>
sudo systemctl restart <service>

# Check service status
sudo systemctl status <service>

# Check running processes
ps aux | grep <process_name>

# List files in a directory
ls -lah

# Edit a file with nano or vim
nano <file_name>
vim <file_name>
```

---

## **3. Installing NGINX**

### **On Ubuntu**

```bash
sudo apt update
sudo apt install nginx -y
```

### **On Docker**

```bash
docker run --name my-nginx -p 80:80 -d nginx
```

After installation, check the status:

```bash
systemctl status nginx
```

Enable NGINX to start on boot:

```bash
sudo systemctl enable nginx
```

---

## **4. NGINX Configuration Structure**

### **Main Configuration File:**

- Located at `/etc/nginx/nginx.conf`
- Includes global settings, worker processes, and event handling.

### **Events Block:**

Handles worker connections:

```nginx
worker_connections 1024;
```

### **HTTP Block:**

Contains settings like gzip compression, MIME types, and virtual hosts.

```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript;
```

- **Why MIME Types?**
  - MIME types tell the browser how to handle different file formats.

## 4. Understanding Nginx Configuration Structure

Nginx configuration files are included in `nginx.conf` and follow a modular approach:

- **Main configuration file**: `/etc/nginx/nginx.conf`
- **Virtual host configs**: `/etc/nginx/sites-available/` (stored) & `/etc/nginx/sites-enabled/` (enabled via symlink)
- **Modular configs**: `/etc/nginx/conf.d/` (directly included in `nginx.conf`)
- **MIME types**: Defined in `/etc/nginx/mime.types` to ensure correct file handling.

### Including Modular Configurations in `nginx.conf`

**File: `/etc/nginx/nginx.conf`**

```nginx
http {
    include /etc/nginx/mime.types;  # Defines how file types are served
    include /etc/nginx/conf.d/*.conf;  # Loads additional configurations
    include /etc/nginx/sites-enabled/*;  # Loads enabled sites
}
```

---

## **5. Hosting a Static Website (React App Hosting)**

Create a virtual host file:

```bash
sudo nano /etc/nginx/sites-available/react-app
```

Add the following:

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/react-app;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

Enable the site:

```bash
ln -s /etc/nginx/sites-available/react-app /etc/nginx/sites-enabled/
nginx -t
sudo systemctl reload nginx
```

### **Fixing `/about` Not Working**

Modify `location /`:

```nginx
try_files $uri $uri/ /index.html;
```

---

## **6. Reverse Proxy with NGINX**

Instead of exposing app ports directly:

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## **7. Load Balancing in NGINX**

### **Using Round-Robin Load Balancing**

```nginx
upstream backend {
    server server1.example.com;
    server server2.example.com;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

### **Weighted Load Balancing**

```nginx
upstream backend {
    server server1.example.com weight=3;
    server server2.example.com weight=1;
}
```

### **Backup & Down Servers**

```nginx
upstream backend {
    server server1.example.com;
    server server2.example.com backup;
    server server3.example.com down;
}
```

---

## Using conf.d for a Modular Approach

- Instead of modifying the main nginx.conf, you can create multiple configuration files inside the /etc/nginx/conf.d/ directory.

- This keeps configurations modular and easier to manage

### Static Site Hosting

- You can create individual config files for different subdomains like:
  - _/etc/nginx/conf.d/cafe.kabir.com.conf_ → Serves cafe.kabir.com
  - /etc/nginx/conf.d/profile.kabir.com.conf → Serves profile.kabir.com
- Each config file can specify different root directories and settings for serving different static sites.

### Load Balancer and Reverse Proxy:

- we can make **/etc/nginx/conf.d/api_proxy.conf, /etc/nginx/conf.d/load_balancer.conf** files like this and write seperate configuration for more modular approach.

## **8. Authentication with Basic Auth**

### **Creating Authentication File**

```bash
sudo apt install apache2-utils
htpasswd -c /etc/nginx/.htpasswd user1
```

### **Enabling Basic Authentication**

```nginx
location /admin {
    auth_basic "Restricted Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

### **Disabling Auth for Specific Routes**

```nginx
location /public {
    auth_basic off;
}
```

**Why Use Basic Auth?**

- Adds an extra layer of security for admin pages.
- Prevents unauthorized access.

---

## **9. TLS/SSL and HTTPS Security**

### **Why Use SSL?**

- Prevents **Man-in-the-Middle (MITM)** attacks.
- Encrypts requests & responses.

### **How SSL Works**

1. **Public/Private Key Encryption Issue**
   - If only a public key is used, an attacker can replace it.
2. **SSL Solution**
   - A signature is created using the server’s public key and a Certificate Authority (CA) public key.
   - The client verifies this signature to ensure authenticity.

### **Setting Up Free SSL (Let's Encrypt)**

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com
```

Automatic renewal:

```bash
sudo certbot renew --dry-run
```

---

## **10. HTTP Caching with NGINX**

```nginx
location /static/ {
    expires 30d;
    add_header Cache-Control "public, max-age=2592000";
}
```

---

## **11. Important NGINX Commands**

```bash
# Test configuration
sudo nginx -t

# Reload NGINX without downtime
sudo systemctl reload nginx

# Restart NGINX
sudo systemctl restart nginx

# Stop NGINX
sudo systemctl stop nginx

# Start NGINX
sudo systemctl start nginx

# Graceful shutdown
nginx -s stop

# Reload configurations without restarting
nginx -s reload
```

---

## **12. Additional Use Cases for NGINX**

- Web Application Firewall (WAF)
- Streaming media (HLS, DASH)
- API Gateway
- Rate Limiting

This guide should provide you with a comprehensive reference for using **NGINX** efficiently. 🚀
