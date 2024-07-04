Comprehensive guide to installing and configuring WordPress with NGINX and ModSecurity as a Web Application Firewall (WAF) on Alpine Linux.
### Update the System
1. Edit repositories:
   ```sh
   nano /etc/apk/repositories
   ```
   Add the following line:
   ```sh
   http://dl-cdn.alpinelinux.org/alpine/latest-stable/community
   ```
2. Update and upgrade the system:
   ```sh
   apk update
   apk upgrade
   ```

### Install Required Packages
```sh
apk add nginx mariadb mariadb-client php82 php82-fpm php82-opcache php82-gd php82-mysqli php82-zlib php82-curl php82-mbstring php82-json php82-session php82-xml gcc g++ make automake libtool pcre pcre-dev zlib zlib-dev libxml2 libxml2-dev curl-dev m4 autoconf automake libtool git geoip geoip-dev yajl yajl-dev lmdb lmdb-dev lua5.3 lua5.3-dev pcre2 pcre2-dev linux-headers
```
Enable services to start at boot:
```sh
rc-update add nginx
rc-update add mariadb
rc-update add php-fpm82
```
Start the services:
```sh
service nginx start
service php-fpm82 start
service mariadb setup
service mariadb start
```
Secure the MariaDB installation:
```sh
mysql_secure_installation
```

### Create a Database and User for WordPress
1. Log in to MariaDB:
   ```sh
   mysql -u root -p
   ```
2. Create the database and user:
   ```sql
   CREATE DATABASE wordpress;
   CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

### Install WordPress
1. Download and set up WordPress:
   ```sh
   wget https://wordpress.org/latest.tar.gz
   tar -xzvf latest.tar.gz
   mv wordpress /var/www/
   chown -R nginx:nginx /var/www/wordpress
   ```
2. Configure PHP-FPM:
   ```sh
   nano /etc/php82/php-fpm.d/www.conf
   ```
   Ensure the following settings:
   ```sh
   user = nginx
   group = nginx
   listen = /var/run/php82-fpm.sock
   listen.owner = nginx
   listen.group = nginx
   listen.mode = 0660
   ```

### Create an NGINX Server Block for Your Domain
1. Create a new configuration file:
   ```sh
   nano /etc/nginx/http.d/waf.codepromax.com.de.conf
   ```
2. Add the following configuration:
   ```nginx
   server {
       listen 80;
       server_name waf.codepromax.com.de;

       root /var/www/wordpress;
       index index.php index.html index.htm;

       access_log /var/log/nginx/waf.codepromax.com.de.access.log;
       error_log /var/log/nginx/waf.codepromax.com.de.error.log;

       # Load ModSecurity configuration
       modsecurity on;
       modsecurity_rules_file /etc/nginx/modsec/modsecurity.conf;

       location / {
           try_files $uri $uri/ /index.php?$args;
       }

       location ~ \.php$ {
           include fastcgi_params;
           fastcgi_pass unix:/var/run/php82-fpm.sock;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           fastcgi_index index.php;
       }

       location ~ /\.ht {
           deny all;
       }
   }
   ```

### Enable and Reload NGINX
1. Test the NGINX configuration:
   ```sh
   nginx -t
   ```
2. Reload NGINX to apply changes:
   ```sh
   nginx -s reload
   ```

### Obtain and Install SSL Certificate (Optional but Recommended)
1. Install Certbot:
   ```sh
   apk add certbot certbot-nginx
   ```
2. Obtain an SSL certificate:
   ```sh
   certbot --nginx -d waf.codepromax.com.de
   ```
3. Follow the prompts to complete the SSL installation.

### Update NGINX Server Block for SSL (If SSL is installed)
1. Edit the NGINX configuration file for your domain:
   ```sh
   nano /etc/nginx/http.d/waf.codepromax.com.de.conf
   ```
2. Update the configuration:
   ```nginx
   server {
       listen 80;
       server_name waf.codepromax.com.de;
       return 301 https://$host$request_uri;
   }

   server {
       listen 443 ssl;
       server_name waf.codepromax.com.de;

       ssl_certificate /etc/letsencrypt/live/waf.codepromax.com.de/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/waf.codepromax.com.de/privkey.pem;
       include /etc/letsencrypt/options-ssl-nginx.conf;
       ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

       root /var/www/wordpress;
       index index.php index.html index.htm;

       access_log /var/log/nginx/waf.codepromax.com.de.access.log;
       error_log /var/log/nginx/waf.codepromax.com.de.error.log;

       # Load ModSecurity configuration
       modsecurity on;
       modsecurity_rules_file /etc/nginx/modsec/modsecurity.conf;

       location / {
           try_files $uri $uri/ /index.php?$args;
       }

       location ~ \.php$ {
           include fastcgi_params;
           fastcgi_pass unix:/var/run/php82-fpm.sock;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           fastcgi_index index.php;
       }

       location ~ /\.ht {
           deny all;
       }
   }
   ```
3. Test the NGINX configuration:
   ```sh
   nginx -t
   ```
4. Reload NGINX to apply the changes:
   ```sh
   nginx -s reload
   ```

### Verify Your Setup
Open your browser and navigate to `http://waf.codepromax.com.de` or `https://waf.codepromax.com.de`.

### Install PCRE2 from Source
1. Download the PCRE2 source code:
   ```sh
   cd /home/erp/
   wget https://github.com/PhilipHazel/pcre2/releases/download/pcre2-10.42/pcre2-10.42.tar.gz
   tar -xzf pcre2-10.42.tar.gz
   cd pcre2-10.42
   ```
2. Build and install PCRE2:
   ```sh
   ./configure
   make
   make install
   ```

### Install LMDB from Source
1. Download the LMDB source code:
   ```sh
   cd /home/erp/
   git clone https://github.com/LMDB/lmdb.git
   cd lmdb/libraries/liblmdb
   ```
2. Build and install LMDB:
   ```sh
   make
   make install
   ```

### Install ssdeep from Source
1. Download the ssdeep source code:
   ```sh
   cd /home/erp/
   wget https://github.com/ssdeep-project/ssdeep/releases/download/release-2.14.1/ssdeep-2.14.1.tar.gz
   tar -xzf ssdeep-2.14.1.tar.gz
   cd ssdeep-2.14.1
   ```
2. Build and install ssdeep:
   ```sh
   ./configure
   make
   make install
   ```

### Proceed with ModSecurity Build
1. Download the ModSecurity source code:
   ```sh
   cd /home/erp/
   git clone https://github.com/SpiderLabs/ModSecurity
   cd ModSecurity
   git submodule init
   git submodule update
   ```
2. Run the build script:
   ```sh
   ./build.sh
   ```
3. Configure and compile ModSecurity:
   ```sh
   ./configure
   make
   make install
   ```

### Install ModSecurity-nginx Connector and NGINX
1. Download the ModSecurity-nginx connector:
   ```sh
   cd /home/erp/
   git clone https://github.com/SpiderLabs/ModSecurity-nginx
   cd ModSecurity-nginx
   ```
2. Download the NGINX source code:
   ```sh
   cd /home/erp/
   nginx -v
   wget http://nginx.org/download/nginx-<version>.tar.gz
   tar -xzvf nginx-<version>.tar.gz
   cd nginx-<version>
   ```
3. Compile NGINX with the ModSecurity-nginx connector:
   ```sh
   ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx
   make modules
   cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules
   ```

4. Edit the NGINX configuration file to load ModSecurity:
   ```nginx
   load_module modules/ngx_http_modsecurity_module.so;
   ```

### Download and Configure the ModSecurity Rules and OWASP Core Rule Set
1. Create the ModSecurity configuration directory:
   ```sh
   mkdir -p /etc/nginx/modsec/rules
  

 ```
2. Download and set up the OWASP Core Rule Set:
   ```sh
   cd /etc/nginx/modsec/rules
   wget https://github.com/coreruleset/coreruleset/archive/v4.0.0.tar.gz
   tar -xzvf v4.0.0.tar.gz
   mv coreruleset-4.0.0/crs-setup.conf.example crs-setup.conf
   mv coreruleset-4.0.0/rules/* .
   ```
3. Create the ModSecurity configuration file:
   ```sh
   nano /etc/nginx/modsec/modsecurity.conf
   ```
   Add the following content:
   ```sh
   Include /usr/local/modsecurity/etc/modsecurity.conf
   SecRuleEngine On
   Include /etc/nginx/modsec/rules/crs-setup.conf
   Include /etc/nginx/modsec/rules/*.conf
   ```

4. Edit the ModSecurity main configuration:
   ```sh
   nano /usr/local/modsecurity/etc/modsecurity.conf
   ```
   Ensure the following settings:
   ```sh
   SecRuleEngine On
   SecRequestBodyAccess On
   SecResponseBodyAccess On
   ```

### Reload NGINX
1. Test the NGINX configuration:
   ```sh
   nginx -t
   ```
2. Reload NGINX to apply the changes:
   ```sh
   nginx -s reload
   ```

### Final Steps
1. Verify the WordPress installation by visiting `http://waf.codepromax.com.de`.
2. Complete the WordPress setup through the web interface.
3. Ensure ModSecurity is active and properly configured by checking the logs for any blocked or allowed requests.

This guide should now provide a clearer path to setting up WordPress with NGINX and ModSecurity on Alpine Linux. Let me know if you need any further assistance!
