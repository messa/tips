Let's Encrypt
=============


Essential links
---------------

https://letsencrypt.org/

https://wiki.debian.org/LetsEncrypt


Debian installation
-------------------

I recommend the [certbot](https://packages.debian.org/search?keywords=certbot) package

### Debian stretch

Install from [stretch-backports](https://backports.debian.org/Instructions/)

```shell
$ sudo apt install -t stretch-backports certbot
```

Nginx manual setup
------------------

```shell
$ domain=www.example.com
$ sudo mkdir -p /srv/www/$domain
$ sudo vim /etc/nginx/sites-enabled/$domain
```

Modify the http (port 80) section - add location `/.well-known/acme-challenge`:

```
server {
    listen 80;
    server_name www.example.com;
    location / {
        ...
    }
    location '/.well-known/acme-challenge' {
        default_type "text/plain";
        root /srv/www/www.example.com;
  }
```

```shell
$ sudo systemctl reload nginx.service
```

Now setup Let's Encrypt using the certbot client:

```shell
$ sudo certbot certonly -d $domain --webroot --webroot-path /srv/www/$domain
... answer all the questions and check the output at the end:
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/www.example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/www.example.com/privkey.pem
   Your cert will expire on 2019-02-25. ...
```

Enable the certificate in nginx configuration:

```shell
$ sudo vim /etc/nginx/sites-enabled/$domain
```

Add or modify the https (port 443) section:

```
server {
    listen 80;
    ...
}
server {
    listen 443;
    server_name www.example.com;
    ssl on;
    ssl_certificate     /etc/letsencrypt/live/www.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.example.com/privkey.pem;
    ...
}
```

And reload nginx:

```shell
$ sudo systemctl reload nginx.service
```
