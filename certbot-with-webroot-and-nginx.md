# Installing Certbot using Webroot and Nginx

The following installs a new Certbot instance on a machine to only acquire
and renew certificates and not mess with Nginx configuration. 

* Install the Debian package `certbot`.

* Set up the Nginx sites for which you want to get certificates to include
  the following location:

  ```
  location /.well-known/acme-challenge/ {
      alias /var/run/acme/webroot/.well-known/acme-challenge/;
  }
  ```

  If you have more than one site, consider sticking it into a snippet and
  include it via

  ```
  include /etc/nginx/snippets/letsencrypt.conf;
  ```

  You only need this for plain-text HTTP, but it can’t hurt to also have
  it on 443.

* Create the webroot directory:

  ```
  mkdir /var/run/acme/webroot
  ```

  It can stay owned as root?

* Initialize Certbot and get your first certificate via

  ```
  certbot certonly --webroot -w /var/run/acme/webroot/ -d $(DOMAIN)
  ```

  The `-d` can be given multiple times. The first one will serve as the
  subject.

* Prepare the SSL version of the sites.

  For each server, include these three lines, replacing `<CERT_NAME>`
  with the name of certificate, i.e., the subject from the previous step:

  ```
  ssl_certificate /etc/letsencrypt/live/<CERT_NAME>/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/<CERT_NAME>/privkey.pem;
  ssl_trusted_certificate /etc/letsencrypt/live/<CERT_NAME>/chain.pem;
  ```

  (The trusted certificate is so Nginx can do OCSP stapling.)

  In addition, the following configures SSL properly. Once again, for 
  multiple sites, you probably want to put this into a snippet and include
  it.

  ```
  listen 443 ssl http2;
  listen [::]:443 ssl http2;

  ssl_session_timeout 1d;
  ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
  ssl_session_tickets off;

  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers off;

  # HSTS (ngx_http_headers_module is required) (63072000 seconds)
  add_header Strict-Transport-Security "max-age=63072000" always;

  # OCSP stapling
  ssl_stapling on;
  ssl_stapling_verify on;
  ```

* Add a renewal hook to Certbot. Add the following script as
  `/etc/letsencrypt/renewal-hooks/deploy/01-reload-nginx`:

  ```
  #! /usr/bin
  systemctl reload nginx
  ```

  Make it executable.

* You can test renewal via `certbot renew --force-renewal`.

