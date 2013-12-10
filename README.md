# Errbit on Nginx

Guide for people to setup their own errbit without using heroku.

### The open source, self-hosted error catcher

  * Install MongoDB. Follow the directions [here](http://www.mongodb.org/display/DOCS/Ubuntu+and+Debian+packages), then:

```bash
apt-get update
apt-get install mongodb-10gen
```

  * Install libxml and libcurl

```bash
apt-get install libxml2 libxml2-dev libxslt-dev libcurl4-openssl-dev
```

  * Install Bundler

```bash
gem install bundler
```

Deploying:
----------

  * Copy `config/deploy.example.rb` to `config/deploy.rb`
  * Update the `deploy.rb` or `config.yml` file with information about your server
  * Setup server and deploy

```bash
cap deploy:setup deploy db:create_mongoid_indexes
```

(Note: The capistrano deploy script will automatically generate a unique secret token.)

It will deploy your app into /var/www/apps/errbit/

# Nginx configuration

Setup Nginx by doing 

  * apt-get install nginx

  * sudo useradd -s /sbin/nologin -r nginx
  * sudo usermod -a -G web nginx
  * sudo chgrp -R web /var/www
  * sudo chmod -R 775 /var/www 

# Sample nginx.conf

  * user nginx;
  * worker_processes 4;
  * pid /run/nginx.pid;  * 

  * events {
  *   worker_connections 1024;
  *   # multi_accept on;
  * }  * 

  * http {  * 

  *   ##
  *   # Basic Settings
  *   ##  * 

  *   sendfile on;
  *   tcp_nopush on;
  *   tcp_nodelay on;
  *   keepalive_timeout 65;
  *   types_hash_max_size 2048;
  *   # server_tokens off;  * 

  *   # server_names_hash_bucket_size 64;
  *   # server_name_in_redirect off;  * 

  *   include /etc/nginx/mime.types;
  *   default_type application/octet-stream;  * 

  *   ##
  *   # Logging Settings
  *   ##  * 

  *   access_log /var/log/nginx/access.log;
  *   error_log /var/log/nginx/error.log;  * 

  *   ##
  *   # Gzip Settings
  *   ##  * 

  *   gzip on;
  *   gzip_disable "msie6";  * 

  *   # gzip_vary on;
  *   # gzip_proxied any;
  *   # gzip_comp_level 6;
  *   # gzip_buffers 16 8k;
  *   # gzip_http_version 1.1;
  *   # gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;  * 

  *   ##
  *   # nginx-naxsi config
  *   ##
  *   # Uncomment it if you installed nginx-naxsi
  *   ##  * 

  *   #include /etc/nginx/naxsi_core.rules;  * 

  *   ##
  *   # nginx-passenger config
  *   ##
  *   # Uncomment it if you installed nginx-passenger
  *   ##
  *   
  *   #passenger_root /usr;
  *   #passenger_ruby /usr/bin/ruby;  * 

  *   ##
  *   # Virtual Host Configs
  *   ##  * 

  *   include /etc/nginx/conf.d/*.conf;
  *   include /etc/nginx/sites-enabled/*;  * 

  * }

# Modify sites-enabled

Find default at /etc/nginx/sites-enabled/ and modify with the following

 *   upstream unicorn_server {
 *    # This is the socket we configured in unicorn.rb
 *    server unix:/var/www/apps/errbit/current/tmp/sockets/unicorn.sock
 *    fail_timeout=0;
 *   }
 * 

 * server {
 *         listen 80 default_server;
 *         listen [::]:80 default_server ipv6only=on;

 *         root /var/www/apps/errbit/current/public;
 *         client_max_body_size 4G;
 *         # Make site accessible from http://localhost/
 *         server_name (your server);
 *         keepalive_timeout 5;
 *         
 *         location / {
 *                 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 *                 proxy_set_header Host $http_host;
 *                 proxy_redirect off; * 

 *                 # If you don't find the filename in the static files
 *                 # Then request it from the unicorn server
 *                 if (!-f $request_filename) {
 *                         proxy_pass http://unicorn_server;
 *                         break;
 *                 }
 *         }
 * }
