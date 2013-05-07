## 170 Install Nginx

Nginx is a web router that is coupled with

```console
rvmsudo passenger-install-nginx-module
```

`<enter>`

```console
Enter your choice (1 or 2) or press Ctrl-C to abort: 1

Please specify a prefix directory [/opt/nginx]: /usr/local/nginx
```


Edit `sudo vi /usr/local/nginx/conf/nginx.conf`

```
http {
  passenger_root /usr/local/rvm/gems/ruby-2.0.0-p0/gems/passenger-4.0.2;
  passenger_ruby /usr/local/rvm/wrappers/ruby-2.0.0-p0/ruby;

  # Allow for up to 10 megabyte uploads
  include       mime.types;
  default_type  application/octet-stream;
  client_max_body_size 10m;

  # Enable GZIP compression
  gzip on;
  gzip_http_version 1.0;
  gzip_vary on;
  gzip_comp_level 6;
  gzip_proxied any;
  gzip_types text/plain text/css application/json application/javascript application/x-javascript text/javascript;
  proxy_set_header Accept-Encoding  "";
  gzip_buffers 16 8k;
  # Disable gzip for certain browsers.
  gzip_disable "MSIE [1-6].(?!.*SV1)";

  server {
    listen      80;
    server_name _;
    rewrite     ^(.*)   https://epiproXX.dipr.partners.org$1 permanent;
  }

  server {
    listen      443;
    server_name _;

    ssl                  on;
    ssl_certificate      cert.pem;
    ssl_certificate_key  cert.key;

    ssl_session_timeout  5m;

    ssl_protocols  SSLv2 SSLv3 TLSv1;
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers   on;

    # rack_env production; # This is the default, could be changed to development and uncommented.

    root  /var/www/html;   # <--- For rails app on subdomains
    passenger_enabled on;
    # Symbolic links in /var/www/html that point to the PUBLIC directory of the rails apps.
    # Ex: rails app at /usr/local/rails_app1
    # sudo ln -s /usr/local/rails_app1/public /var/www/html/rails_app1
    # passenger_base_uri /rails_app1;
    # passenger_base_uri /rails_app2;
  }
}
```


### 172 Apache - Install Phusion Passenger (Web server)

If you don't want Nginx, you can use Apache instead

```
passenger-install-apache2-module
```

Add the provided lines to `/etc/apache2/other/rails.conf`

```
LoadModule passenger_module /usr/local/rvm/gems/ruby-1.9.3-p194/gems/passenger-3.0.13/ext/apache2/mod_passenger.so
PassengerRoot /usr/local/rvm/gems/ruby-1.9.3-p194/gems/passenger-3.0.13
PassengerRuby /usr/local/rvm/wrappers/ruby-1.9.3-p194/ruby

PassengerMaxPoolSize 2
PassengerPoolIdleTime 360

RackEnv production
PassengerHighPerformance on

<VirtualHost *:80>
  RewriteEngine On
  RewriteCond %{HTTPS} off
  RewriteRule (.*) https://myexternalurl.com%{REQUEST_URI}
</VirtualHost>
```

Restart Apache (or equivalent nginx command)

```console
sudo apachectl restart
```

### Next Step

[75 - Nginx Automatic Startup](https://github.com/remomueller/documentation/tree/master/macosx/75-nginx-automatic-startup-configuration.rdoc)