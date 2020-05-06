MongoDB 4.2 TLS Replica Set Cluster Setup
=========================================


Step 1. Prepare VMs
-------------------

Let's use **DigitalOcean** Droplets for that.
[Get **$100 free DigitalOcean credit** with this referral link.](https://m.do.co/c/389daec654bc)

You can use any cloud provider you want, or your own hardware, with or without virtualization.
I just think DigitalOcean is the easiest way how to get a new VM working.

👉 Create three droplets. Use FQDN as their hostname, use a domain you own so the hostnames really work.
I will use `testNN.example.com` here as a placeholder. Select a Debian 10 image and a datacenter convenient for you.

- `test01.example.com`
- `test02.example.com`
- `test03.example.com`


Step 2. Setup DNS records for the VMs
-------------------------------------

👉 Setup domain records so that the hostnames translate to the correct IP address.

For example if you host DNS on AWS Route53 and use Terraform to manage the AWS resources:

```terraform
resource "aws_route53_record" "A_w-test01_example_com" {
  zone_id = data.aws_route53_zone.messa_cz.zone_id
  name    = "test01.example.com"
  type    = "A"
  ttl     = "1800"
  records = ["10.20.30.111"]
}

resource "aws_route53_record" "A_w-test02_example_com" {
  zone_id = data.aws_route53_zone.messa_cz.zone_id
  name    = "test02.example.com"
  type    = "A"
  ttl     = "1800"
  records = ["10.20.30.222"]
}

resource "aws_route53_record" "A_w-test03_example_com" {
  zone_id = data.aws_route53_zone.messa_cz.zone_id
  name    = "test03.example.com"
  type    = "A"
  ttl     = "1800"
  records = ["10.20.30.333"]
}
```


Step 3. Setup the VMs, install MongoDB
--------------------------------------

🔎 For more info how to install MongoDB on Debian: https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/

I recommend to prepare an empty working directory for this and further steps.

👉 Save this script as a file `setup01.sh`:

```shell
#!/bin/bash
set -ex
for hn in $hostnames; do
   ssh root@$hn apt-get update
   ssh root@$hn apt-get upgrade
   ssh root@$hn apt-get install -y --no-install-recommends gnupg2 vim rsync

   curl -sS https://www.mongodb.org/static/pgp/server-4.2.asc | \
     ssh root@$hn apt-key add -

   echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/4.2 main" | \
     ssh root@$hn tee /etc/apt/sources.list.d/mongodb-org-4.2.list

   ssh root@$hn apt-get update
   ssh root@$hn apt-get install mongodb-org-server mongodb-org-shell mongodb-org-tools
done
```

👉 Run it this way:

```shell
$ hostnames="test01.example.com test02.example.com test03.example.com" bash setup01.sh
```


Step 4. Prepare SSL certificates for the MongoDB cluster
--------------------------------------------------------

🔎 MongoDB docs: [Use x.509 Certificate for Membership Authentication](https://docs.mongodb.com/manual/tutorial/configure-x509-member-authentication/)

‼️ You can get a TLS ("SSL") certificate for free from [Let's Encrypt](https://letsencrypt.org/).
But **you should not use Let's Encrypt certificates for MongoDB cluster authentication** – because anybody, who can also use the same CA and obtain certificate with the same DN fields, which in case of Let's Encrypt is *everybody*, can join your cluster and replicate all your data.
I suppose you do not want that.
So we are going to create **our own CA** (certificate authority) that we will use for MongoDB cluster member authentication and for encryption of their communication.

TODO: Makefile etc.


Step 5. Prepare MongoDB configuration file `mongod.conf`
--------------------------------------------------------




```
demo-rs:PRIMARY> use admin
switched to db admin
demo-rs:PRIMARY> db.createUser({ user: 'root', pwd: 'topsecret', roles: ['root'] })
```


```
demo-rs:PRIMARY> rs.add('test02.example.com:27017')
demo-rs:PRIMARY> rs.add('test03.example.com:27017')
```

