# windoac/docker-remote-api-tls

forked from [kekru/docker-remote-api-tls](https://github.com/kekru/docker-remote-api-tls)

All of the setting should be same. Just add this for ARM support.

## Changed

- Add ARM support
- Auto generate `CREATE_CERTS_WITH_PW` if not set
- `CERT_EXPIRATION_DAYS` & `CA_EXPIRATION_DAYS` default is 3650 days

## Quick use
```yml
version: "3.8"

volumes:
  certs:
# add your volume setting here

services:
  remote-api:
    image: windoac/docker-remote-api-tls:latest
    ports:
      - 2376:443
    environment:
      - CERT_HOSTNAME=remote-api.example.com
    volumes:
      - certs:/data/certs
      - /var/run/docker.sock:/var/run/docker.sock:ro
```

## debug

```bash
# build
docker build -t docker-remote-api-tls .

# test run
docker run -it --rm docker-remote-api-tls
```

# from forked kekru/docker-remote-api-tls

# Docker Remote API with TLS client authentication via container

This images makes you publish your Docker Remote API by a container.
A client must authenticate with a client-TLS certificate.
This is an alternative way, instead of [configuring TLS on Docker directly](https://gist.github.com/kekru/974e40bb1cd4b947a53cca5ba4b0bbe5).

[![dockeri.co](https://dockerico.blankenship.io/image/kekru/docker-remote-api-tls)](https://hub.docker.com/r/kekru/docker-remote-api-tls)

## Remote Api with external CA, certificates and key

First you need a CA and certs and keys for your Docker server and the client.

Create them as shown here [Protect the Docker daemon socket](https://docs.docker.com/engine/security/https/).
Or create the files with this script [create-certs.sh](https://github.com/kekru/linux-utils/blob/master/cert-generate/create-certs.sh). Read [Create certificate files](https://gist.github.com/kekru/974e40bb1cd4b947a53cca5ba4b0bbe5#create-certificate-files) for information on how to use the script.

Copy the following files in a directory. The directory will me mounted in the container.

```bash
ca-cert.pem
server-cert.pem
server-key.pem
```

The files `cert.pem` and `key.pem` are certificate and key for the client. The client will also need the `ca-cert.pem`.

Create a docker-compose.yml file:

```yml
version: "3.4"
services:
  remote-api:
    image: kekru/docker-remote-api-tls:v0.5.0
    ports:
     - 2376:443
    volumes:
     - <local cert dir>:/data/certs:ro
     - /var/run/docker.sock:/var/run/docker.sock:ro
```

Now run the container with `docker-compose up -d` or `docker stack deploy --compose-file=docker-compose.yml remoteapi`.
Your Docker Remote API is available on port 2376 via https. The client needs to authenticate via `cert.pem` and `key.pem`.

## Remote Api with auto generating CA, certificates and keys

The docker-remote-api image can generate CA, certificates and keys for you automatically.
Create a docker-compose.yml file, specifying a password and the hostname, on which the remote api will be accessible later on. The hostname will be written to the server's certificate.

```yml
version: "3.4"
services:
  remote-api:
    image: kekru/docker-remote-api-tls:v0.5.0
    ports:
     - 2376:443
    environment:
     - CREATE_CERTS_WITH_PW=supersecret
     - CERT_HOSTNAME=remote-api.example.com
    volumes:
     - <local cert dir>:/data/certs
     - /var/run/docker.sock:/var/run/docker.sock:ro
```

Now run the container with `docker-compose up -d` or `docker stack deploy --compose-file=docker-compose.yml remoteapi`.
Certificates will be created in `<local cert dir>`.
You will find the client-certs in `<local cert dir>/client/`. The files are `ca.pem`, `cert.pem` and `key.pem`.

## Environment variables

#### `CREATE_CERTS_WITH_PW`
Passphrase to encrypt the certificate.

#### `CERTS_PASSWORD_FILE`
Certificate passphrase will be read from this docker secret. Absolute path of the secret file has to be provided i.e. `CERTS_PASSWORD_FILE=/run/secrets/<secret_name>`.

If both passphrase and secret file are set, the secret file takes precedence.

#### `CERT_EXPIRATION_DAYS`
Certificate expiration for server and client certs in days. If not set, the default value 365 is applied.

#### `CA_EXPIRATION_DAYS`
Certificate expiration for CA in days. If not set, the default value 900 is applied.

#### `CERT_HOSTNAME`
Domain name of the docker server.  
If you don't have a DNS name, you can use [nip.io](https://nip.io) to get a name for any IP.  

## Setup client

See [Run commands on remote Docker host](https://gist.github.com/kekru/4e6d49b4290a4eebc7b597c07eaf61f2) for instructions how to setup a client to communicate with the remote api.

You can also reuse [dockerRemote](./dockerRemote) and set url and path in it to your correct values.  
Then just run `./dockerRemote ps` to call `ps` against your remote api.

## Quick test

To test this repo quickly, clone this repo, then run

```bash
# Start remote-api locally
docker-compose up -d
# Run ps over remote api (use GitBash when you are on Windows)
./dockerRemote ps
```

## Changelog

#### v0.2.0

First stable release  
Thanks [@smiller171](https://github.com/smiller171) for contributing!

#### v0.3.0

+ update nginx version
+ add configuration for cert expiration
+ add configuration to use swarm secret as password for cert generation
+ add automatic tests

Thanks [@benkorichard](https://github.com/benkorichard) for contributing!

#### v0.4.0

+ update nginx version to 1.20.2

#### v0.5.0

+ update nginx version to 1.26.2
