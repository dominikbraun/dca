## Repository client verification

Docker runs via a non-networked Unix socket by default. TLS must be enabled in order to have the Docker client and the
daemon securely over HTTPS. TLS ensures authenticity of the registry endpoint and that traffic to/from registry is
encrypted.

### Understand the configuration

A custom certificate is configured by creating a directory under `/etc/docker/certs.d` using the same name as the
registry's hostname, such as `localhost`. All `.crt` files are added to this directory as CA roots.

The presence of one or more `FILENAME.key/cert` pairs indicates to Docker that there are custom certificates required
for access to the desired repository.

The following illustrates a configuration with custom certificates:

```shell script
/etc/docker/certs.d/        <-- Certificate directory
└── localhost:5000          <-- Hostname:port
   ├── client.cert          <-- Client certificate
   ├── client.key           <-- Client key
   └── ca.crt               <-- Certificate authority that signed
                                the registry certificate
```

### Create the client certificates

Use OpenSSL's `genrsa` and `req` commands to first generate an RSA key and then use the key to create the certificate.

```shell script
$ openssl genrsa -out client.key 4096
$ openssl req -new -x509 -text -key client.key -out client.cert
```

### Troubleshooting tips

The Docker daemon interprets `.crt` files as CA certificates and `.cert` files as client certificates. If a CA
certificate is accidentally given the extension `.cert` instead of the correct `.crt` extension, the Docker daemon logs
the following error message:

```shell script
Missing key KEY_NAME for client certificate CERT_NAME. CA certificates should use the extension .crt.
```

If the Docker registry is accessed without a port number, do not add the port to the directory name. The following shows
the configuration for a registry on default port 443 which is accessed with
`docker login my-https.registry.example.com`:

```shell script
/etc/docker/certs.d/
└── my-https.registry.example.com          <-- Hostname without port
   ├── client.cert
   ├── client.key
   └── ca.crt
```