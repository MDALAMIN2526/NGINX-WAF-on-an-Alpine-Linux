# NGINX-WAF-on-an-Alpine-Linux
install Wordpress with NGINX & NGINX ModSecurity as a Web Application Firewall (WAF) on an Alpine Linux
### Update the System
```sh
apk update
```
```sh
apk upgrade
```
```sh
apk add nano
```
```sh
nano /etc/apk/repositories
```
```sh
http://dl-cdn.alpinelinux.org/alpine/latest-stable/community
```

### Install Required Packages

```sh
apk add nano nginx mariadb mariadb-client php82 php82-fpm php82-opcache php82-gd php82-mysqli php82-zlib php82-curl php82-mbstring php82-json php82-session php82-xml gcc g++ make automake libtool pcre pcre-dev zlib zlib-dev libxml2 libxml2-dev curl-dev m4 autoconf automake libtool git geoip geoip-dev yajl yajl-dev lmdb lmdb-dev lua5.3 lua5.3-dev pcre2 pcre2-dev linux-headers
```
```sh
rc-update add nginx
```
```sh
rc-update add mariadb
```
```sh
rc-update add php-fpm82
```
```sh
service nginx start
```
```sh
service php-fpm82 start
```
```sh
/etc/init.d/mariadb setup
```

```sh
service mariadb start
```
```sh
rc-update add mariadb
```
```sh
mysql_secure_installation
```
### Create a database and user for WordPress:
```sh
mysql -u root -p
```
```sh
CREATE DATABASE wordpress;
```
```sh
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
```
```sh
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
```
```sh
FLUSH PRIVILEGES;
```
```sh
EXIT;
```
### Install WordPress
1. **Download the latest WordPress::**
```sh
wget https://wordpress.org/latest.tar.gz
```
```sh
tar -xzvf latest.tar.gz
```
```sh
mv wordpress /var/www/
```
```sh
chown -R nginx:nginx /var/www/wordpress
```
2. **Configure PHP-FPM:**
```sh
nano /etc/php7/php-fpm.d/www.conf
```
4. *Ensure the following settings are configured:*
```sh
user = nginx
group = nginx
listen = /var/run/php7-fpm.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660

```
### Create an NGINX Server Block for Your Domain
Create a new server block configuration file for your domain.

1. **Create a new configuration file:**
   ```sh
   nano /etc/nginx/http.d/waf.codepromax.com.de.conf
   ```

2. **Add the following configuration:**
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
1. **Test the NGINX configuration:**
   ```sh
   nginx -t
   ```

2. **Reload NGINX to apply the changes:**
   ```sh
   nginx -s reload
   ```

### Obtain and Install SSL Certificate (Optional but Recommended)
Using Let's Encrypt to secure your site with SSL:

1. **Install Certbot:**
   ```sh
   apk add certbot certbot-nginx
   ```

2. **Obtain an SSL certificate:**
   ```sh
   certbot --nginx -d waf.codepromax.com.de
   ```

3. **Follow the prompts to complete the SSL installation.**

### Update NGINX Server Block for SSL (If SSL is installed)
1. **Edit the NGINX configuration file for your domain:**
   ```sh
   nano /etc/nginx/http.d/waf.codepromax.com.de.conf
   ```

2. **Update the configuration to redirect HTTP to HTTPS and listen on port 443:**
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

3. **Test the NGINX configuration:**
   ```sh
   nginx -t
   ```

4. **Reload NGINX to apply the changes:**
   ```sh
   nginx -s reload
   ```

### Verify Your Setup
1. **Open your browser and navigate to `http://waf.codepromax.com.de` or `https://waf.codepromax.com.de`.**


### Install `PCRE2` from Source

1. **Download the `PCRE2` source code:**
   ```sh
   cd /home/erp/
   ```
   ```sh
   wget https://github.com/PhilipHazel/pcre2/releases/download/pcre2-10.42/pcre2-10.42.tar.gz
   ```
   ```sh
   tar -xzf pcre2-10.42.tar.gz
   ```
   ```sh
   cd pcre2-10.42
   ```

3. **Build and install `PCRE2`:**

   ```sh
   ./configure
   ```
   ```sh
   make
   ```
   ```sh
   make install
   ```
### Install `LMDB` from Source

1. **Download the `LMDB` source code:**
   ```sh
   cd /home/erp/
   ```
   ```sh
   git clone https://github.com/LMDB/lmdb.git
   ```
   ```sh
   cd lmdb/libraries/liblmdb
   ```

2. **Build and install `LMDB`:**
   
   ```sh
   make
   ```
   ```sh
   make install
   ```
### Install `ssdeep` from Source

1. **Download the `ssdeep` source code:**
   ```sh
   cd /home/erp/
   ```
   ```sh
   wget https://github.com/ssdeep-project/ssdeep/releases/download/release-2.14.1/ssdeep-2.14.1.tar.gz
   ```
   ```sh
   tar -xzf ssdeep-2.14.1.tar.gz
   ```
   ```sh
   cd ssdeep-2.14.1
   ```

2. **Build and install `ssdeep`:**

   ```sh
   ./configure
   ```
   ```sh
   make
   ```
   ```sh
   make install
   ```

### Proceed with ModSecurity Build

1. **Download the ModSecurity source code:**
   ```sh
   cd /home/erp/
   ```
   ```sh
   git clone https://github.com/SpiderLabs/ModSecurity
   ```
   ```sh
   cd ModSecurity
   ```
   ```sh
   git submodule init
   ```
   ```sh
   git submodule update
   ```

2. **Run the build script:**

   ```sh
   ./build.sh
   ```

3. **Configure and compile ModSecurity:**

   ```sh
   ./configure
   ```
   ```sh
   make
   ```
   ```sh
   make install
   ```

### Install ModSecurity-nginx Connector and NGINX

1. **Download the ModSecurity-nginx connector:**
   ```sh
   cd /home/erp/
   ```
   ```sh
   git clone https://github.com/SpiderLabs/ModSecurity-nginx
   ```
     ```sh
   cd ModSecurity-nginx
   ```

2. **Download the NGINX source code:**
   ```sh
   cd /home/erp/
   ```
   ```sh
   nginx -v
   ```
   ```sh
   wget http://nginx.org/download/nginx-<version>.tar.gz
   ```
   ```sh
   tar -xzvf nginx-<version>.tar.gz

   ```
   ```sh
   cd nginx-<version>
   ```

4. **Compile NGINX with the ModSecurity-nginx connector:**

   ```sh
   ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx
   ```
   ```sh
   make modules
   ```
   ```sh
    cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules
   ```

5. **Edit the NGINX configuration file to load ModSecurity:**

   ```nginx
   load_module modules/ngx_http_modsecurity_module.so;
   ```

### Download and Configure the ModSecurity Rules and OWASP Core Rule Set

1. **Create the ModSecurity configuration directory:**

   ```sh
   mkdir -p /etc/nginx/modsec/rules
   ```

2. **Download the OWASP Core Rule Set:**

   ```sh
   cd /etc/nginx/modsec
   git clone https://github.com/coreruleset/coreruleset.git
   cp -r coreruleset/rules/* rules/
   cp coreruleset/crs-setup.conf.example crs-setup.conf
   ```

3. **Download the ModSecurity configuration file:**

   ```sh
   wget https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/modsecurity.conf-recommended -O modsecurity.conf
   wget https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/unicode.mapping
   ```

4. **Edit the ModSecurity configuration file (`/etc/nginx/modsec/modsecurity.conf`):**

   Enable ModSecurity:

   ```nginx
   SecRuleEngine On
   ```

   Add the following line to include the OWASP CRS setup configuration:

   ```nginx
   Include /etc/nginx/modsec/crs-setup.conf
   ```

   Add the following line to include the OWASP CRS rules:

   ```nginx
   Include /etc/nginx/modsec/rules/*.conf
   ```

### Update NGINX Configuration

1. **Edit the main NGINX configuration file (`/etc/nginx/nginx.conf`) to load the ModSecurity module:**

   ```nginx
   user nginx;
   
   # Set number of worker processes automatically based on number of CPU cores.
   worker_processes auto;
   
   # Enables the use of JIT for regular expressions to speed-up their processing.
   pcre_jit on;
   
   # Configures default error logger.
   error_log /var/log/nginx/error.log warn;
   
   # Includes files with directives to load dynamic modules.
   include /etc/nginx/modules/*.conf;
   
   # Load the ModSecurity module
   load_module modules/ngx_http_modsecurity_module.so;
   
   # Include files with config snippets into the root context.
   include /etc/nginx/conf.d/*.conf;
   
   events {
       # The maximum number of simultaneous connections that can be opened by
       # a worker process.
       worker_connections 1024;
   }
   
   http {
       # Includes mapping of file name extensions to MIME types of responses
       # and defines the default type.
       include /etc/nginx/mime.types;
       default_type application/octet-stream;
   
       # Name servers used to resolve names of upstream servers into addresses.
       # It's also needed when using tcpsocket and udpsocket in Lua modules.
       #resolver 1.1.1.1 1.0.0.1 [2606:4700:4700::1111] [2606:4700:4700::1001];
   
       # Don't tell nginx version to the clients. Default is 'on'.
       server_tokens off;
   
       # Specifies the maximum accepted body size of a client request, as
       # indicated by the request header Content-Length. If the stated content
       # length is greater than this size, then the client receives the HTTP
       # error code 413. Set to 0 to disable. Default is '1m'.
       client_max_body_size 1m;
   
       # Sendfile copies data between one FD and other from within the kernel,
       # which is more efficient than read() + write(). Default is off.
       sendfile on;
   
       # Causes nginx to attempt to send its HTTP response head in one packet,
       # instead of using partial frames. Default is 'off'.
       tcp_nopush on;
   
       # Enables the specified protocols. Default is TLSv1 TLSv1.1 TLSv1.2.
       # TIP: If you're not obligated to support ancient clients, remove TLSv1.1.
       ssl_protocols TLSv1.2 TLSv1.3;
   
       # Path of the file with Diffie-Hellman parameters for EDH ciphers.
       # TIP: Generate with: `openssl dhparam -out /etc/ssl/nginx/dh2048.pem 2048`
       #ssl_dhparam /etc/ssl/nginx/dh2048.pem;
   
       # Specifies that our cipher suits should be preferred over client ciphers.
       # Default is 'off'.
       ssl_prefer_server_ciphers on;
   
       # Enables a shared SSL cache with size that can hold around 8000 sessions.
       # Default is 'none'.
       ssl_session_cache shared:SSL:2m;
   
       # Specifies a time during which a client may reuse the session parameters.
       # Default is '5m'.
       ssl_session_timeout 1h;
   
       # Disable TLS session tickets (they are insecure). Default is 'on'.
       ssl_session_tickets off;
   
       # Enable gzipping of responses.
       #gzip on;
   
       # Set the Vary HTTP header as defined in the RFC 2616. Default is 'off'.
       gzip_vary on;
   
       # Helper variable for proxying websockets.
       map $http_upgrade $connection_upgrade {
           default upgrade;
           '' close;
       }
   
       # Specifies the main log format.
       log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                       '$status $body_bytes_sent "$http_referer" '
                       '"$http_user_agent" "$http_x_forwarded_for"';
   
       # Sets the path, format, and configuration for a buffered log write.
       access_log /var/log/nginx/access.log main;
   
       # Includes virtual hosts configs.
       include /etc/nginx/http.d/*.conf;
      }
   ```


### Verify and Start NGINX

1. **Verify your NGINX configuration:**

   ```sh
   nginx -t
   ```

2. **If the test is successful, start and enable NGINX:**

   ```sh
   rc-service nginx start
   rc-update add nginx
   ```

3. **Reload NGINX to apply changes:**

   ```sh
   nginx -s reload
   ```

4. **Check NGINX logs for any ModSecurity-related issues:**

   ```sh
   tail -f /var/log/nginx/error.log
   ```

This completes the installation and configuration of NGINX with ModSecurity as a Web Application Firewall (WAF) on Alpine Linux.
