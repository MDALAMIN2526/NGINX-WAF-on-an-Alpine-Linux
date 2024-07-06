Here's a step-by-step guide to setting up an Ubuntu 22.04 server with NGINX, configuring a WordPress site (wap.codepromax.com.de), a Laravel v10 site (wap2.codepromax.com.de), and setting up NGINX ModSecurity:

### Update the System
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Necessary Prerequisites
```bash
sudo apt install nginx php-fpm php-mysql php-xml php-mbstring php-curl php-zip mysql-server git gcc build-essential libtool libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev -y
```

### Configer MySQL Server
```bash
sudo mysql_secure_installation
```
### Download and Compile ModSecurity
```bash
cd /usr/local/src
```
```bash
sudo git clone --depth 1 https://github.com/SpiderLabs/ModSecurity
```
```bash
cd ModSecurity
```
```bash
sudo git submodule init
```
```bash
sudo git submodule update
```
```bash
sudo ./build.sh
```
```bash
sudo ./configure
```
```bash
sudo make
```
```bash
sudo make install
```

### Download and Compile ModSecurity NGINX Connector
```bash
cd /usr/local/src
```
```bash
sudo git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
```

### Download and Compile NGINX with ModSecurity
First, download the NGINX source code. Make sure to get the version matching your currently installed NGINX.
```bash
wget http://nginx.org/download/nginx-1.18.0.tar.gz
```
```bash
tar -zxvf nginx-1.18.0.tar.gz
```
```bash
cd nginx-1.18.0
```

### Compile NGINX with the ModSecurity module:
```bash
sudo ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx
```
```bash
sudo make modules
```

### Move the compiled module to the NGINX modules directory:
```bash
sudo mkdir -p /etc/nginx/modules
```
```bash
sudo cp /usr/local/src/nginx-1.18.0/objs/ngx_http_modsecurity_module.so /etc/nginx/modules/

```

### Configure NGINX to Load ModSecurity Module
Edit the NGINX configuration to load the ModSecurity module:
```bash
sudo nano /etc/nginx/nginx.conf
```
Add the following line at the top:
```nginx
load_module modules/ngx_http_modsecurity_module.so;
```

### Configure ModSecurity
Move the ModSecurity configuration file to the appropriate directory:
```bash
sudo mv /usr/local/src/ModSecurity/modsecurity.conf-recommended /etc/nginx/modsecurity.conf
```
```bash
sudo mv /usr/local/src/ModSecurity/unicode.mapping /etc/nginx/
```

### Edit the ModSecurity configuration file to enable the engine:
```bash
sudo nano /etc/nginx/modsecurity.conf
```
```bash
sudo nginx -t
```
Change `SecRuleEngine DetectionOnly` to `SecRuleEngine On`.

### Download OWASP Core Rule Set
```bash
sudo git clone https://github.com/coreruleset/coreruleset /etc/nginx/owasp-crs
```
```bash
sudo cp /etc/nginx/owasp-crs/crs-setup.conf.example /etc/nginx/owasp-crs/crs-setup.conf
```
### Include the CRS configuration in your ModSecurity configuration:
```bash
echo 'Include /etc/nginx/owasp-crs/crs-setup.conf' | sudo tee -a /etc/nginx/modsecurity.conf
```
```bash
echo 'Include /etc/nginx/owasp-crs/rules/*.conf' | sudo tee -a /etc/nginx/modsecurity.conf
```

### Create Databases for WordPress and Laravel
```bash
sudo mysql -u root -p
```
```bash
CREATE DATABASE wordpress_db;
```
```bash
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'strong_password';
```
```bash
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
```
```bash
FLUSH PRIVILEGES;
```
```bash
CREATE DATABASE laravel_db;
```
```bash
CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'strong_password';
```
```bash
GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
```
```bash
FLUSH PRIVILEGES;
```
```bash
EXIT;
```

### Download and Setup WordPress
```bash
cd /var/www
```
```bash
sudo wget https://wordpress.org/latest.tar.gz
```
```bash
sudo tar -xzvf latest.tar.gz
```
```bash
sudo mv wordpress wap.codepromax.com.de
```
```bash
sudo chown -R www-data:www-data /var/www/wap.codepromax.com.de
```
```bash
sudo chmod -R 755 /var/www/wap.codepromax.com.de
```

### Setup Laravel
```bash
cd /var/www
```
```bash
sudo apt install composer -y
```
```bash
sudo composer create-project --prefer-dist laravel/laravel wap2.codepromax.com.de
```
```bash
sudo chown -R www-data:www-data /var/www/wap2.codepromax.com.de
```
```bash
sudo chmod -R 755 /var/www/wap2.codepromax.com.de
```

### Configure NGINX for WordPress and Laravel
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
```
```bash
sudo ln -s /etc/nginx/sites-available/wap2.codepromax.com.de /etc/nginx/sites-enabled/
```
```bash
sudo chown -R www-data:www-data /var/log/nginx
```
```bash
sudo chmod -R 755 /var/log/nginx
```
```bash
sudo systemctl restart nginx
```
```bash
sudo nginx -t

```

### Update Site Configurations to Use ModSecurity
For WordPress:
```bash
sudo nano /etc/nginx/sites-available/wap.codepromax.com.de
```
Add the following lines within the server block:
```nginx
modsecurity on;
modsecurity_rules_file /etc/nginx/modsecurity.conf;
```

For Laravel:
```bash
sudo nano /etc/nginx/sites-available/wap2.codepromax.com.de
```
Add the following lines within the server block:
```nginx
modsecurity on;
modsecurity_rules_file /etc/nginx/modsecurity.conf;
```

### Restart NGINX
```bash
sudo systemctl restart nginx
```
By following these steps, NGINX should now correctly serve your applications with ModSecurity enabled to enhance security against potential web attacks. Adjust paths and settings according to your specific server setup and requirements.
