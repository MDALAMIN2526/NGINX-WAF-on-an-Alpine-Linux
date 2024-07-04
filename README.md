# NGINX-WAF-on-an-Alpine-Linux
NGINX with ModSecurity as a Web Application Firewall (WAF) on an Alpine Linux
### Update the System

```sh
apk update
   ```
```sh
apk upgrade
   ```
### Step 1: Install Required Packages

```sh
apk add nginx gcc g++ make automake libtool pcre pcre-dev zlib zlib-dev libxml2 libxml2-dev curl-dev m4 autoconf automake libtool git geoip geoip-dev yajl yajl-dev lmdb lmdb-dev lua5.3 lua5.3-dev pcre2 pcre2-dev linux-headers
```
 the `ssdeep` and `ssdeep-dev` packages are not available in the default Alpine Linux repositories. We can build and install `ssdeep` from source instead. Here are the updated steps:

### Step 2: Install `PCRE2` from Source

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
### Step 3: Install `LMDB` from Source

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
### Step 3: Install `ssdeep` from Source

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

### Step 4: Proceed with ModSecurity Build

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

### Step 5: Install ModSecurity-nginx Connector and NGINX

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

6. **Configure ModSecurity in NGINX:**

   ```nginx
   server {
       listen 80;
       server_name example.com;

       modsecurity on;
       modsecurity_rules_file /etc/nginx/modsec/modsecurity.conf;

       location / {
           proxy_pass http://backend_server;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

7. **Download and configure the ModSecurity rules and the OWASP Core Rule Set (optional).**

8. **Start and enable NGINX:**

   ```sh
    rc-service nginx start
   ```
   ```sh
    rc-update add nginx
   ```

9. **Test the configuration:**

   ```sh
    nginx -t
   ```
   ```sh
    nginx -s reload
   ```

This should complete the installation and configuration of NGINX with ModSecurity as a Web Application Firewall (WAF) on Alpine Linux.

