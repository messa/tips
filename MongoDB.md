
MongoDB
=======

MongoDB is a document-oriented database.

Features:

- replication (_replica sets_) that is pretty easy to setup and maintain
- sharding (with multiple replica sets accessed from _mongos_)
- aggregation framework; I don't use it, but it's there
- plug-in engine architecture, two engines are currently included: MMAPv1 and WiredTiger


Production setup
----------------

Resources:

- [Production Notes](http://docs.mongodb.org/manual/administration/production-notes/)
- [Production Checklist](http://docs.mongodb.org/manual/administration/production-checklist/)
- [Security Checklist](http://docs.mongodb.org/manual/administration/security-checklist/)
- [Configure `mongod` and `mongos` for TLS/SSL](http://docs.mongodb.org/manual/tutorial/configure-ssl/)
- [Deploy Replica Set and Configure Authentication and Authorization](http://docs.mongodb.org/manual/tutorial/deploy-replica-set-with-auth/)
- [MongoDB, TLS, and x.509 Authentication Deep Dive (allanbank.com)](http://www.allanbank.com/blog/security/tls/x.509/2014/10/13/tls-x509-and-mongodb/)

Following notes are for Debian Jessie and MongoDB version 3.0 (current at this time).


### Installation

Follow this: http://docs.mongodb.org/manual/tutorial/install-mongodb-on-debian/

- add `deb` record to `/etc/apt/sources.list.d/mongodb-org-3.0.list`
- `sudo aptitude update`
- `sudo aptitude install mongodb-org`


### A few notes about SSL

Communication will be encrypted using SSL.
Setup SSL CA, for example as it is described in
[OpenSSL.md](https://github.com/messa/tips/blob/master/OpenSSL.md#make-your-own-ca).

MongoDB distinguishes two kinds of SSL (x.509) certificates:

- _Member certificates_
- _Client certificates_

#### Member certificates

([MongoDB docs: Use x.509 Certificate for Membership Authentication](http://docs.mongodb.org/manual/tutorial/configure-x509-member-authentication/#member-x-509-certificate))

The member certificate, used for internal authentication to verify membership
to the sharded cluster or a replica set, must have the following properties:

- A single Certificate Authority (CA) must issue all the x.509 _member_ certificates

- The Distinguished Name (DN), found in the member certificate’s subject,
  must specify a non-empty value for at least one of the following attributes:

    - Organization (`O`),
    - the Organizational Unit (`OU`) or
    - the Domain Component (`DC`).

- `O`, `OU` and `DC` must _match_ those from the certificates for the other cluster members

- Either the Common Name (`CN`) or one of the Subject Alternative Name (`SAN`) entries
  must match the hostname of the server, used by the other members of the cluster.

For example, the certificates for a cluster could have the following subjects:

    subject= CN=<myhostname1>,OU=Dept1,O=MongoDB,ST=NY,C=US
    subject= CN=<myhostname2>,OU=Dept1,O=MongoDB,ST=NY,C=US
    subject= CN=<myhostname3>,OU=Dept1,O=MongoDB,ST=NY,C=US

If you use Extended Key Usage (EKU) attributes:

- The certificate that you specify for the `PEMKeyFile` option requires the `serverAuth` attribute,
- the certificate you specify to `clusterFile` requires the `clientAuth` attribute.

If you omit `clusterFile`, mongod will use the certificate specified to PEMKeyFile for member authentication,
which can be a problem if it doesn't have the `clientAuth` attribute.


#### Client certificates

([MongoDB docs: Use x.509 Certificates to Authenticate Clients](http://docs.mongodb.org/manual/tutorial/configure-x509-client-authentication/#client-x-509-certificate))

The client certificate must have the following properties:

- A single Certificate Authority (CA) must issue the certificates for both the client and the server.
- Client certificates must contain the following fields:

        keyUsage = digitalSignature
        extendedKeyUsage = clientAuth

- Each unique MongoDB user must have a unique certificate.
- A client x.509 certificate’s subject, which contains the Distinguished Name (DN), must _differ_ from that of a Member x.509 Certificate, with regards to at least one of the following attributes:

    - Organization (`O`),
    - the Organizational Unit (`OU`) or
    - the Domain Component (`DC`).

    If a client x.509 certificate’s subject has the same O, OU, and DC combination as the Member x.509 Certificate, the client will be identified as a cluster member and _granted full permission_ on the system.

### SSL CA setup




### Configuration


Create directory `/etc/mongo-ssl`.

Configuration file `/etc/mongodb.conf`:

    TODO

Start MongoDB:

    sudo /etc/init.d/mongod restart

See `/var/log/mongodb/mongod.log`


### Initiate replica set

Run `mongo` and check replica set status.

    > rs.status()
    {
        "info" : "run rs.initiate(...) if not yet done for the set",
        "ok" : 0,
        "errmsg" : "no replset config has been received",
        "code" : 94

That's OK.

    > rs.initiate()
    {
        "info2" : "no configuration explicitly specified -- making one",
        "me" : "exampleserver.example.com:27017",
        "ok" : 1
    }

Check status again:

    lhprod1:PRIMARY> rs.status()
    {
        "set" : "lhprod1",
        "date" : ISODate("2015-06-09T13:49:36.869Z"),
        "myState" : 1,
        "members" : [
            {
                "_id" : 0,
                "name" : "exampleserver.example.com:27017",
                "health" : 1,
                "state" : 1,
                "stateStr" : "PRIMARY",
                "uptime" : 215,
                "optime" : Timestamp(1433857746, 1),
                "optimeDate" : ISODate("2015-06-09T13:49:06Z"),
                "electionTime" : Timestamp(1433857746, 2),
                "electionDate" : ISODate("2015-06-09T13:49:06Z"),
                "configVersion" : 1,
                "self" : true
            }
        ],
        "ok" : 1
    }

Check replica set configuration:

    lhprod1:PRIMARY> rs.conf()
    2015-06-09T13:51:19.786+0000 E QUERY    Error: Could not retrieve replica set config: {
        "ok" : 0,
        "errmsg" : "not authorized on admin to execute command { replSetGetConfig: 1.0 }",
        "code" : 13
    }
        at Function.rs.conf (src/mongo/shell/utils.js:1017:11)
        at (shell):1:4 at src/mongo/shell/utils.js:1017

We need to create admin user.


### Create admin user

    lhprod1:PRIMARY> use admin
    switched to db admin

    lhprod1:PRIMARY> db.createUser({user: 'admin', pwd: 'topsecret', roles: [
                     {role: "userAdminAnyDatabase", db: "admin"},
                     {role: "root", db: "admin"}]})

    Successfully added user: {
        "user" : "admin",
        "roles" : ...
    }



Links
-----

- https://www.mongodb.org/
- http://docs.mongodb.org/manual/faq/
- http://api.mongodb.org/python/current/
- https://aphyr.com/posts/284-call-me-maybe-mongodb :)

