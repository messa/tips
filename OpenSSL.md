
OpenSSL cheatsheet
==================

Make your own CA
----------------

For example for MongoDB deployment with SSL, or IPSec.

Resources:

- https://jamielinux.com/docs/openssl-certificate-authority/index.html


### Prepare root and intermediate CA

__Prepare directory:__

    mkdir ca
    cd ca
    mkdir certs crl newcerts private
    chmod 700 private
    touch index.txt
    echo 1000 > serial

__Create configuration file__ `openssl.conf` (in the `ca` directory):

    [ ca ]
    default_ca = CA_default

    [ CA_default ]
    dir               = /XXX/ca
    certs             = $dir/certs
    crl_dir           = $dir/crl
    new_certs_dir     = $dir/newcerts
    database          = $dir/index.txt
    serial            = $dir/serial
    RANDFILE          = $dir/private/.rand

    # The root key and root certificate.
    private_key       = $dir/private/ca.key.pem
    certificate       = $dir/certs/ca.cert.pem

    # For certificate revocation lists.
    crlnumber         = $dir/crlnumber
    crl               = $dir/crl/ca.crl.pem
    crl_extensions    = crl_ext
    default_crl_days  = 30

    default_md        = sha256

    name_opt          = ca_default
    cert_opt          = ca_default
    default_days      = 36500
    preserve          = no
    policy            = policy_strict

    [ policy_strict ]
    # The root CA should only sign intermediate certificates that match.
    # See the POLICY FORMAT section of `man ca`.
    countryName             = match
    stateOrProvinceName     = match
    organizationName        = match
    organizationalUnitName  = optional
    commonName              = supplied
    emailAddress            = optional

    [ policy_loose ]
    # Allow the intermediate CA to sign a more diverse range of certificates.
    # See the POLICY FORMAT section of the `ca` man page.
    countryName             = optional
    stateOrProvinceName     = optional
    localityName            = optional
    organizationName        = optional
    organizationalUnitName  = optional
    commonName              = supplied
    emailAddress            = optional

    [ req ]
    default_bits        = 2048
    distinguished_name  = req_distinguished_name
    string_mask         = utf8only

    default_md          = sha256

    # Extension to add when the -x509 option is used.
    x509_extensions     = v3_ca

    [ req_distinguished_name ]
    # See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
    countryName                     = Country Name (2 letter code)
    stateOrProvinceName             = State or Province Name
    localityName                    = Locality Name
    0.organizationName              = Organization Name
    organizationalUnitName          = Organizational Unit Name
    commonName                      = Common Name
    emailAddress                    = Email Address

    # Optionally, specify some defaults.
    countryName_default             = GB
    stateOrProvinceName_default     = England
    localityName_default            =
    0.organizationName_default      = Alice Ltd
    organizationalUnitName_default  =
    emailAddress_default            =

    [ v3_ca ]
    # Extensions for a typical CA (`man x509v3_config`).
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid:always,issuer
    basicConstraints = critical, CA:true
    keyUsage = critical, digitalSignature, cRLSign, keyCertSign

    [ v3_intermediate_ca ]
    # Extensions for a typical intermediate CA (`man x509v3_config`).
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid:always,issuer
    basicConstraints = critical, CA:true, pathlen:0
    keyUsage = critical, digitalSignature, cRLSign, keyCertSign

    [ usr_cert ]
    # Extensions for client certificates (`man x509v3_config`).
    basicConstraints = CA:FALSE
    nsCertType = client, email
    nsComment = "OpenSSL Generated Client Certificate"
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid,issuer
    keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
    extendedKeyUsage = clientAuth, emailProtection

    [ server_cert ]
    # Extensions for server certificates (`man x509v3_config`).
    basicConstraints = CA:FALSE
    nsCertType = server
    nsComment = "OpenSSL Generated Server Certificate"
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid,issuer:always
    keyUsage = critical, digitalSignature, keyEncipherment
    extendedKeyUsage = serverAuth

    [ crl_ext ]
    # Extension for CRLs (`man x509v3_config`).
    authorityKeyIdentifier=keyid:always

    [ ocsp ]
    # Extension for OCSP signing certificates (`man ocsp`).
    basicConstraints = CA:FALSE
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid,issuer
    keyUsage = critical, digitalSignature
    extendedKeyUsage = critical, OCSPSigning

This configuration file template is copied from [jamielinux.com](https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html),
there is also description of the configuration options.

- __Update `dir`__ in the `[ca_default]` section to correct value.

- Maybe you would like to update `default_crl_days` in `[ca_default]`.

- I also recommend to set `countryName_default` and other defaults to your
  preferred values.

__Create the root key__ (all the following operations are still in the `ca` directory):

    openssl genrsa -aes256 -out private/ca.key.pem 4096
    # it will ask for password

    chmod 400 private/ca.key.pem

__Create the root certificate:__

    openssl req \
      -config openssl.conf \
      -key private/ca.key.pem \
      -new -x509 -days 36500 -sha256 -extensions v3_ca \
      -out certs/ca.cert.pem
    # it will ask for the root key password and for root certificate details;
    # enter something like 'Alice Ltd Root CA' as Common Name

    chmod 444 certs/ca.cert.pem

Verify the root certificate:

    openssl x509 -noout -text -in certs/ca.cert.pem

__Create the intermediate pair:__

    mkdir intermediate # so the full path will be .../ca/intermediate
    cd intermediate
    mkdir certs crl csr newcerts private
    chmod 700 private
    touch index.txt
    echo 1000 > serial
    echo 1000 > crlnumber
    cp ../openssl.conf .

Change these options in the `ca/intermediate/openssl.conf`:

    [ CA_default ]
    dir             = /XXX/ca/intermediate
    private_key     = $dir/private/intermediate.key.pem
    certificate     = $dir/certs/intermediate.cert.pem
    crl             = $dir/crl/intermediate.crl.pem
    policy          = policy_loose

Create the intermediate key:

    cd .. # so the full path will be .../ca
    openssl genrsa -aes256 \
      -out intermediate/private/intermediate.key.pem 4096
    # it will ask for intermediate key password

    chmod 400 intermediate/private/intermediate.key.pem

Create the intermediate certificate:

    openssl req \
      -config intermediate/openssl.conf \
      -new -sha256 \
      -key intermediate/private/intermediate.key.pem \
      -out intermediate/csr/intermediate.csr.pem
    # it will ask for intermediate key password and intermediate
    # certificate details; it must match the root cert details except
    # the Common Name, which I recommend to be something like
    # 'Alice Ltd Intermediate CA'

    openssl ca \
      -config openssl.conf \
      -extensions v3_intermediate_ca \
      -days 36500 -notext -md sha256 \
      -in intermediate/csr/intermediate.csr.pem \
      -out intermediate/certs/intermediate.cert.pem
    # it will ask for root key password;
    # answer yes (y) to 'Sign the certificate?' :)

    chmod 444 intermediate/certs/intermediate.cert.pem

Verify the intermediate certificate:

    openssl x509 -noout -text \
      -in intermediate/certs/intermediate.cert.pem

    openssl verify -CAfile certs/ca.cert.pem \
      intermediate/certs/intermediate.cert.pem

Create the certificate chain file:

    cat \
      intermediate/certs/intermediate.cert.pem \
      certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem

    chmod 444 intermediate/certs/ca-chain.cert.pem

### Create and sign server certificates

Still in the `ca` directory.

    name=www.example.com

Create key:

    openssl genrsa -aes256 \
      -out intermediate/private/$name.key.pem 4096
    # you will be asked for key password

    chmod 400 intermediate/private/$name.key.pem

Create CSR:

    openssl req -config intermediate/openssl.conf \
      -key intermediate/private/$name.key.pem \
      -new -sha256 -out intermediate/csr/$name.csr.pem

Create certificate:

    # use usr_cert instead of server_cert if the certificate
    # is going to be used on client

    openssl ca -config intermediate/openssl.conf \
      -extensions server_cert -days 36500 -notext -md sha256 \
      -in intermediate/csr/$name.csr.pem \
      -out intermediate/certs/$name.cert.pem

    chmod 444 intermediate/certs/www.example.com.cert.pem

Verify the certificate:

    openssl x509 -noout -text \
      -in intermediate/certs/$name.cert.pem

    openssl verify -CAfile intermediate/certs/ca-chain.cert.pem \
      intermediate/certs/$name.cert.pem

When deploying to a server application,
you need to make the following files available:

- `ca-chain.cert.pem`
- `$name.key.pem`
- `$name.cert.pem`


Links
-----

- waiting for https://letsencrypt.org/ :)



