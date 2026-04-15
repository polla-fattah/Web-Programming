---
theme: academic
title: "Deployment & Optimization"
info: |
  VPS • Permissions • SSL/TLS • OPcache
paginate: true
---

# Server Deployment & Optimization (Week 13)
## Bridging Localhost and the Global Web

---

## 1. The Deployment Paradigm Shift
For 12 weeks, your application has existed strictly on `localhost:8000`. Deploying an application means moving this codebase to a remotely accessible computer (Server) connected directly to the internet via a static **Public IP Address**.

* **The Local Stack:** You relied on `php artisan serve` for routing and local XAMPP/Docker for MySQL.
* **The Production Stack:** `artisan serve` is strictly banned in production. A true server requires an enterprise Web Server (Nginx or Apache) to actively listen on Port 80 (HTTP) and Port 443 (HTTPS), acting as a highly-concurrent traffic manager.

---

## 2. Server Architectures
As software engineers, you must evaluate where your application lives.

* **Shared Hosting (cPanel):** The legacy, inexpensive option (often used for student projects). 
  * *Pros:* Everything is pre-installed.
  * *Cons:* No root access. You physically share underlying server resources with hundreds of other websites. Highly restrictive for modern frameworks.
* **Virtual Private Server (VPS):** The enterprise standard (DigitalOcean, AWS EC2, Linode). 
  * *Pros:* Complete `root` Linux access. Dedicated RAM and CPU segments. Highly scalable.
  * *Cons:* Zero graphical interface (Terminal/SSH only). You must manually secure the firewall and install Nginx, PHP-FPM, and PostgreSQL yourself.

---

## 3. Directory Structure Security (The #1 Vulnerability)
The most common and catastrophic mistake junior developers make occurs when configuring the Nginx/Apache Web Server Root.

**The Catastrophic Vulnerability:**
If you point your domain (e.g., `application.com`) to the `/laravel-app` root folder, attackers can type `application.com/.env` into their browser and instantly download your database passwords and API keys!

**The Architectural Fix:**
Modern PHP frameworks are designed with a single public entry point. You must strictly configure the web server root to point to `/laravel-app/public`. This ensures all sensitive backend controllers and `.env` files remain securely trapped *behind* the firewall, completely unreachable by web browsers.

---

## 4. The Production `.env` File
The `.env` file must never be pushed to Git. When you clone your code to the remote server, you must manually recreate `.env` and switch the system into Production Mode.

```env
# 1. Securing the Environment
APP_ENV=production

# 2. Preventing Whoops Errors 
# If true, an exception will dump raw SQL queries and stack traces to the user screen!
# If false, Laravel securely shows a generic "500 Server Error" graphic.
APP_DEBUG=false

# 3. Setting the rigorous production domain
APP_URL=https://my-university-project.com

# 4. Connecting to the remote Production Database
DB_HOST=127.0.0.1
DB_DATABASE=production_db
DB_USERNAME=root
DB_PASSWORD=SecureProductionPassword!
```

---

## 5. Securing the Transmission Layer (SSL/TLS)
Modern APIs and session cookies require cryptographic certificates to prevent Man-in-the-Middle (MitM) attacks. Without it, your HTTP traffic is sent in raw plaintext.

**Let's Encrypt & Certbot:**
You do not need to buy expensive SSL certificates anymore. **Let's Encrypt** is a free, open-source certificate authority.

By running a single command via SSH, **Certbot** will automatically generate a valid SSL certificate and reconfigure your Nginx server to forcefully redirect all standard HTTP (Port 80) traffic to secure HTTPS (Port 443).

```bash
# Automatically secures your domain in seconds
sudo certbot --nginx -d my-university-project.com
```

---

## 6. The Deployment Lifecycle (SSH & Git)
Legacy FTP (dragging and dropping files via FileZilla) is highly prone to corruption and takes minutes to upload framework vendor folders. Enterprise deployment relies entirely on `Git`.

**The Manual VPS Update Flow:**
```bash
# 1. Securely connect to the remote Linux server
ssh root@192.168.1.100

# 2. Navigate to your application directory
cd /var/www/my-application

# 3. Pull the absolute latest code directly from GitHub
git pull origin main

# 4. Install any new dependencies (ignoring local dev tools like Pest)
composer install --optimize-autoloader --no-dev

# 5. Migrate any new database schema changes forcefully
php artisan migrate --force
```

---

## 7. The Linux Permission Hurdle (`www-data`)
The command above (`git pull`) is usually executed by the `root` or `ubuntu` user. However, the Nginx web server operates under an entirely different Linux user named **`www-data`**. 

**The Crash:** When a user uploads an avatar, or Laravel attempts to write an error to `laravel.log`, it will crash with a `Permission Denied` error because Nginx does not own the folders!

**The Solution:** Grant explicit ownership and write permissions back to the web server.
```bash
# Granting overall ownership of the application to the web server user
sudo chown -R www-data:www-data /var/www/my-application

# Ensuring specific storage and cache directories are mathematically writable
sudo chmod -R 775 /var/www/my-application/storage
sudo chmod -R 775 /var/www/my-application/bootstrap/cache
```

---

## 8. Process Management (Supervisor & Queues)
Enterprise applications use **Queues** (e.g., sending emails in the background). If you SSH into the server and type `php artisan queue:work`, the queue processes perfectly... until you close your SSH terminal window, at which point the queue dies instantly.

**The Solution: Process Daemons**
Linux utilizes Process Monitors like **Supervisor** to background daemon processes. 

Supervisor continuously watches the `queue:work` process 24/7. If the process crashes due to memory limits or a severe API exception, Supervisor automatically reboots it within seconds, ensuring your background jobs never drop!

---

## 9. Zero-Downtime Automation (Modern Tools)
Manually typing SSH commands, migrating databases, and fixing permissions is tedious and terrifying on Friday evenings. Modern teams utilize automated deployment pipelines.

* **Laravel Forge & Laravel Envoyer:** Official tools that automatically provision Ubuntu servers, configure Nginx, and manage SSL certificates with the click of a button.
* **Zero-Downtime Deployments:** Instead of taking the application offline while `composer install` runs, automation tools clone the new code into a secondary hidden folder, build it completely, and then instantly flip the Nginx symlink. The user experiences absolutely zero downtime!

---

## 10. Performance Optimization (OPCache)
On your laptop, Laravel dynamcally reads hundreds of configuration files and translates PHP into machine code on every single request. In production, this causes massive CPU spikes.

**Framework Caching:**
```bash
php artisan config:cache # Bundles config/* into a single array
php artisan route:cache  # Pre-compiles the routing logic
php artisan view:cache   # Pre-compiles the raw Blade syntax
```

**PHP Level Optimization (OPCache):**
PHP is an interpreted language. **OPCache** is a native PHP extension that stores pre-compiled script bytecode in the server's shared RAM. Combining Laravel's `route:cache` with PHP's OPcache bypasses parsing entirely, allowing the framework to handle thousands of requests per second
