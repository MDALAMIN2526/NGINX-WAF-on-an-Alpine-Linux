Here's a step-by-step guide to setting up an Ubuntu 22.04 server with NGINX, configuring a WordPress site (wap.codepromax.com.de), a Laravel v10 site (wap2.codepromax.com.de), and setting up NGINX ModSecurity:

### Step 1: Update the System
```bash
sudo apt update
sudo apt upgrade -y
```

### Step 2: Install NGINX
```bash
sudo apt install nginx -y
```

### Step 3: Install PHP and Necessary Extensions
```bash
sudo apt install php-fpm php-mysql php-xml php-mbstring php-curl php-zip -y
```

### Step 4: Install MySQL Server
```bash
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

### Step 5: Create Databases for WordPress and Laravel
```bash
sudo mysql -u root -p
CREATE DATABASE wordpress_db;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;

CREATE DATABASE laravel_db;
CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Step 6: Download and Setup WordPress
```bash
cd /var/www
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
sudo mv wordpress wap.codepromax.com.de
sudo chown -R www-data:www-data /var/www/wap.codepromax.com.de
sudo chmod -R 755 /var/www/wap.codepromax.com.de
```

### Step 7: Setup Laravel
```bash
cd /var/www
sudo apt install composer -y
sudo composer create-project --prefer-dist laravel/laravel wap2.codepromax.com.de
sudo chown -R www-data:www-data /var/www/wap2.codepromax.com.de
sudo chmod -R 755 /var/www/wap2.codepromax.com.de
```

### Step 8: Configure NGINX for WordPress and Laravel
Create the NGINX configuration file for WordPress:
```bash
sudo nano /etc/nginx/sites-available/wap.codepromax.com.de
```
Add the following configuration:
```nginx
server {
    listen 80;
    server_name wap.codepromax.com.de;

    root /var/www/wap.codepromax.com.de;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
Create the NGINX configuration file for Laravel:
```bash
sudo nano /etc/nginx/sites-available/wap2.codepromax.com.de
```
Add the following configuration:
```nginx
server {
    listen 80;
    server_name wap2.codepromax.com.de;

    root /var/www/wap2.codepromax.com.de/public;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
Enable the sites and restart NGINX:
```bash
sudo ln -s /etc/nginx/sites-available/wap.codepromax.com.de /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/wap2.codepromax.com.de /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

### Step 9: Install and Configure ModSecurity
```bash
sudo apt install libnginx-mod-http-modsecurity -y
sudo mv /etc/nginx/modsecurity/modsecurity.conf-recommended /etc/nginx/modsecurity/modsecurity.conf
sudo nano /etc/nginx/modsecurity/modsecurity.conf
```
Change `SecRuleEngine DetectionOnly` to `SecRuleEngine On`.

### Step 10: Download OWASP Core Rule Set
```bash
sudo git clone https://github.com/coreruleset/coreruleset /etc/nginx/modsecurity/owasp-crs
sudo cp /etc/nginx/modsecurity/owasp-crs/crs-setup.conf.example /etc/nginx/modsecurity/owasp-crs/crs-setup.conf
```

### Step 11: Configure NGINX to Use ModSecurity
Edit the NGINX configuration files to include ModSecurity:
For WordPress:
```bash
sudo nano /etc/nginx/sites-available/wap.codepromax.com.de
```
Add the following lines within the server block:
```nginx
modsecurity on;
modsecurity_rules_file /etc/nginx/modsecurity/modsecurity.conf;
```

For Laravel:
```bash
sudo nano /etc/nginx/sites-available/wap2.codepromax.com.de
```
Add the following lines within the server block:
```nginx
modsecurity on;
modsecurity_rules_file /etc/nginx/modsecurity/modsecurity.conf;
```
Restart NGINX:
```bash
sudo systemctl restart nginx
```

Now your Ubuntu server is configured with NGINX, serving a WordPress site on wap.codepromax.com.de, a Laravel site on wap2.codepromax.com.de, and NGINX ModSecurity for enhanced security.
