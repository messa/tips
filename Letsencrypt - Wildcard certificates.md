Letsencrypt – Wildcard certificates
===================================

Route53
-------

We will use the recommended client - [certbot](https://certbot.eff.org/).

The Route53 plugin documentation is here: https://certbot-dns-route53.readthedocs.io/

Create the credentials as described in the documentation.

The easiest way how to run certbot is to use Docker:

```shell
$ sudo docker run 
      -it --rm --name certbot \
      -v "/etc/letsencrypt:/etc/letsencrypt" \
      -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
      -v "/path/to/aws/credentials:/root/.aws/config" \
      certbot/dns-route53 \
      certonly \
      -d example.com -d '*.example.com'
```

The certificates need to be renewed - add the command above to `/etc/cron.d/certbot-docker`: 

```
0 */12 * * * root /usr/bin/docker run -t --rm --name certbot-renew -v ... -v ... -v ... certbot/dns-route53 -q renew
```
