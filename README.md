## Dockerised reverse proxy

This repository contains a support for running a `nginx` proxy in a docker container in order to secure a web application.  It makes a number of assumptions:

* You want to secure the app with TLS and redirect all http traffic to https
* You have a single service container to proxy and it is speaking http

The configuration takes as a starting point [`montagu-proxy`](https://github.com/vimc/montagu-proxy), which was adapted for use with [`orderly-web-deploy`](https://github.com/vimc/orderly-web-deploy).

### Configuration

Before starting we need to know what we are proxying (i.e., the name of the container on the docker network that is the main entrypoint) and what the proxy will be seen as to the outside world (the hostname, and ports for http and https).  The entrypoint takes these four values as arguments.

### SSL Certificates

The server will not start until the files `/run/proxy/certificate.pem` and `/run/proxy/key.pem` exist - you can get these into the container however you like; the proxy will poll for them and start within a second of them appearing.

### Self signed certificate

For testing it is useful to use a self-signed certificate.  These are not in any way secure.  To generate a self-signed certificate, there is a utility in the proxy container `self-signed-certificate` that will generate one on demand after receiving key components of the CSR.

There is a self-signed certificate in the repo for testing generated with (on metal)

```
./bin/self-signed-certificate ssl GB London "Imperial College" reside web-dev.dide.ic.ac.uk
```

These can be used in the container by `exec`-ing `self-signed-certificate /run/proxy` in the container while it polls for certificates.  Alternatively, to generate certificates with a custom CSR (which takes a couple of seconds) you can exec

```
self-signed-certificate GB London IC vimc montagu.vaccineimpact.org
```

### `dhparams` (Diffie-Hellman key exchange parameters)

We require a `dhparams.pem` file (see [here](https://security.stackexchange.com/questions/94390/whats-the-purpose-of-dh-parameters) for details.  To regenerate this file, run

```
./bin/dhparams ssl
```

from this directory, commit the result to git and rebuild the containers.  This takes quite a while to run (several minutes).  You can copy your own into the container at `/run/proxy/dhparams.pem` before getting the certificates in place.

### Usage

```
docker run --name proxy reside/proxy-nginx:reside-53 service example.com 80 443
docker cp ssl/certificate.pem proxy:/run/proxy/certificate.pem
docker cp ssl/key.pem proxy:/run/proxy/key.pem
```

## License

MIT © Imperial College of Science, Technology and Medicine
