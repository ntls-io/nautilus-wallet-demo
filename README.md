# Nautilus Wallet demo deployment

This is a Docker Compose based deployment of [nautilus-wallet].

See also: https://github.com/ntls-io/nautilus-wallet/wiki/Deployments

[nautilus-wallet]: https://github.com/ntls-io/nautilus-wallet


## Services

### HTTP ingress: `nginx-proxy`

This runs [nginx-proxy] and exposes ports 80 and 443, forwarding HTTP requests through to the other services (which listen on internal ports).

[nginx-proxy]: https://github.com/nginx-proxy/nginx-proxy

#### Configuration

TLS certificates are mounted from `/etc/nginx/certs` on the host, and the proxied services use `VIRTUAL_HOST` and `CERT_NAME` to configure which host name and certificate should proxy to them.

### Wallet APIs: `wallet-api-*`

These run the individual Wallet TEEs using the host's SGX devices, each with a Docker volume to persist wallet state (`/app/wallet_store`).

### Asset Services: `asset-services-*`

This runs the Asset Services API endpoint, Celery worker, and Redis (Celery broker backend).

The Redis broker persists short-lived state (limited to Celery task queue messages and results) in a Docker volume.

#### Configuration

The Celery worker uses environment variables for credentials and configuration: see the `asset-services` [docker-compose.yaml].

[docker-compose.yaml]: https://github.com/ntls-io/nautilus-wallet/blob/main/asset-services/docker-compose.yaml

### Setting up GitHub Actions & SSH for deployment

In addition to the application configuration, the following three GitHub Actions environment values determine access to the deployment target:

- `DOCKER_HOST`: The Docker target host, in the format `ssh://user@target-host`
- `SSH_KEY`: The SSH private key to connect with
- `SSH_KNOWN_HOSTS`: The target host's SSH public key

A new SSH private key may be generated using a command like:

```
ssh-keygen -t ed25519 -f id-nautilus-wallet-demo -C 'nautilus-wallet-demo GitHub Actions' -N ''
```

The value of `SSH_KNOWN_HOSTS` may be obtained from:

```shell
ssh-keyscan -t ed25519 'target-host'
```
