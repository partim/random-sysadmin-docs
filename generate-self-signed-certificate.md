# Generating a Self-signed Certificate with OpenSSL

The following instructions can be used to generate a self-signed
X.509 server certificate that will be accepted by more strict TLS
implementations for `localhost`.

You will need the `openssl` command.

Unfortunately, you cannot simply use `openssl req -x509 -newkey` as that
will insist on creating a CA certificate which will be rejected by some
TLS implementations even if you add additional extensions. So we have to
do this step by step.

## Generate a Key Pair

```
openssl genrsa -out localhost.key 4096
```

The number at the end is the key length in bits.


## Generate a Certificate Signing Request

```
openssl req -new -nodes -out localhost.csr -key localhost.key \
    -sha256 -subj /CN=localhost -addext "subjectAltName = DNS:localhost" \
    -addext "keyUsage = digitalSignature" \
    -addext "extendedKeyUsage = serverAuth"
```

This will create a certificate signing request including the necessary
extensions. If you want to add more extensions, see `man x509v3_config`.
The argument to `-addext` is one key-value pair from that manual page.


## Generate the Certificate

```
openssl x509 -req -days 365 -in localhost.csr -signkey localhost.key \
    -out localhost.crt -copy_extensions copyall
```

The `-days` option specifies the number of days until the certificate
expires.


## Check the Resulting Certificate

To get a human-readable version of the certificate’s content, run

```
openssl x509 -in localhost.crt -noout -text
```

