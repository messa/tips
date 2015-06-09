
OpenSSL cheatsheet
==================

Make your own CA
----------------

For example for MongoDB deployment with SSL, or IPSec.
Should not be used to create a certificate for a public website.

Resources:

- Inspired by [jamielinux.com](https://jamielinux.com/docs/openssl-certificate-authority/index.html)

Directory structure that will be created:

    ├── intermediate-ca
    │   ├── certs
    │   │   ├── ca-chain.cert.pem
    │   │   ├── ca.cert.pem
    │   │   └── www.example.com.cert.pem
    │   ├── crl
    │   ├── crlnumber
    │   ├── csr
    │   │   ├── ca.csr.pem
    │   │   └── www.example.com.csr.pem
    │   ├── index.txt
    │   ├── index.txt.attr
    │   ├── newcerts
    │   │   └── 1000.pem
    │   ├── openssl.conf
    │   ├── private
    │   │   ├── ca.key.pem
    │   │   └── www.example.com.key.pem
    │   ├── serial
    └── root-ca
        ├── certs
        │   └── ca.cert.pem
        ├── crl
        ├── index.txt
        ├── index.txt.attr
        ├── newcerts
        │   └── 1000.pem
        ├── openssl.conf
        ├── private
        │   └── ca.key.pem
        ├── serial

### Prepare root and intermediate CA

__Prepare directory:__

    mkdir root-ca
    mkdir root-ca/{certs,crl,newcerts,private}
    chmod 700 root-ca/private
    touch root-ca/index.txt
    echo 1000 > root-ca/serial

__Create configuration file__ `root-ca/openssl.conf` from [sample openssl.conf](https://gist.github.com/messa/82cf72cdd51e894c890c).
See [jamielinux.com](https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html)
for configuration options description.

- __Update `dir`__ in the `[ca_default]` section to correct value.

- Maybe you would like to update `default_days` in `[ca_default]`.

- I also recommend to set `countryName_default` and other defaults to your
  preferred values.

__Create the root key__:

    openssl genrsa -aes256 -out root-ca/private/ca.key.pem 4096
    # it will ask for password

    chmod 400 root-ca/private/ca.key.pem

__Create the root certificate:__

    openssl req \
      -config root-ca/openssl.conf \
      -key root-ca/private/ca.key.pem \
      -new -x509 -days 36500 -sha256 -extensions v3_ca \
      -out root-ca/certs/ca.cert.pem
    # it will ask for the root key password and for root certificate details;
    # enter something like 'Alice Ltd Root CA' as Common Name

    chmod 444 root-ca/certs/ca.cert.pem

Verify the root certificate:

    openssl x509 -noout -text -in root-ca/certs/ca.cert.pem

__Create the intermediate pair:__

    mkdir intermediate-ca
    mkdir intermediate-ca/{certs,crl,csr,newcerts,private}
    chmod 700 intermediate-ca/private
    touch intermediate-ca/index.txt
    echo 1000 > intermediate-ca/serial
    echo 1000 > intermediate-ca/crlnumber
    cp root-ca/openssl.conf intermediate-ca/openssl.conf

Change these options in the `intermediate-ca/openssl.conf`:

    [ CA_default ]
    dir             = /XXX/intermediate-ca
    policy          = policy_loose

Create the intermediate key:

    openssl genrsa -aes256 \
      -out intermediate-ca/private/ca.key.pem 4096
    # it will ask for intermediate key password

    chmod 400 intermediate-ca/private/ca.key.pem

Create the intermediate certificate:

    openssl req \
      -config intermediate-ca/openssl.conf \
      -new -sha256 \
      -key intermediate-ca/private/ca.key.pem \
      -out intermediate-ca/csr/ca.csr.pem
    # it will ask for intermediate key password and intermediate
    # certificate details; it must match the root cert details except
    # the Common Name, which I recommend to be something like
    # 'Alice Ltd Intermediate CA'

    openssl ca \
      -config root-ca/openssl.conf \
      -extensions v3_intermediate_ca \
      -days 36500 -notext -md sha256 \
      -in intermediate-ca/csr/ca.csr.pem \
      -out intermediate-ca/certs/ca.cert.pem
    # it will ask for root key password;
    # answer yes (y) to 'Sign the certificate' and 'commit'

    chmod 444 intermediate-ca/certs/ca.cert.pem

Verify the intermediate certificate:

    openssl x509 -noout -text \
      -in intermediate-ca/certs/ca.cert.pem

    openssl verify -CAfile root-ca/certs/ca.cert.pem \
      intermediate-ca/certs/ca.cert.pem

Create the certificate chain file:

    cat \
      intermediate-ca/certs/ca.cert.pem \
      root-ca/certs/ca.cert.pem > intermediate-ca/certs/ca-chain.cert.pem

    chmod 444 intermediate-ca/certs/ca-chain.cert.pem

### Create and sign server certificates

    name=www.example.com

Create key:

    openssl genrsa -aes256 \
      -out intermediate-ca/private/$name.key.pem 4096
    # you will be asked for key password

    chmod 400 intermediate-ca/private/$name.key.pem

Create CSR:

    openssl req -config intermediate-ca/openssl.conf \
      -key intermediate-ca/private/$name.key.pem \
      -new -sha256 -out intermediate-ca/csr/$name.csr.pem
    # it will ask for key password;
    # do not forget to enter Common Name (CN), usually it's hostname

Create certificate:

    # use usr_cert instead of server_cert if the certificate
    # is going to be used on client

    openssl ca -config intermediate-ca/openssl.conf \
      -extensions server_cert -days 36500 -notext -md sha256 \
      -in intermediate-ca/csr/$name.csr.pem \
      -out intermediate-ca/certs/$name.cert.pem

    chmod 444 intermediate-ca/certs/$name.cert.pem

Verify the certificate:

    openssl x509 -noout -text \
      -in intermediate-ca/certs/$name.cert.pem

    openssl verify -CAfile intermediate-ca/certs/ca-chain.cert.pem \
      intermediate-ca/certs/$name.cert.pem

When deploying to a server application,
you need to make the following files available:

- `ca-chain.cert.pem`
- `$name.key.pem`
- `$name.cert.pem`


Links
-----

- waiting for https://letsencrypt.org/ :)



