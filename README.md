<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <img src="assets/icon.svg" alt="authentik logo" width="128">
  </picture>
</p>

<h1 align="center">aac-authentik</h1>

<p align="center">Self-hosted deployment definition for authentik, an open-source identity provider and SSO platform.</p>

<p align="center">
  <a href="https://github.com/goauthentik/authentik"><img alt="Upstream" src="https://img.shields.io/badge/upstream-goauthentik%2Fauthentik-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-ghcr.io%2Fgoauthentik%2Fserver%3A2026.8.5-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/goauthentik/authentik/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-MIT%20(core)-green"></a>
</p>

## About

authentik is a self-hosted identity provider that offers single sign-on (SSO), user and group
management, and multi-factor authentication for web applications. It supports OAuth2/OpenID
Connect, SAML, LDAP and proxy-based authentication, so applications that cannot speak these
protocols natively can still be placed behind a central login.

Key features (as provided by upstream):

- Single sign-on via OAuth2 / OpenID Connect, SAML and LDAP
- Flow-based authentication and enrollment, including MFA
- User, group and application management with a web administration interface
- Outposts (for example LDAP or reverse-proxy integration)
- Email notifications and customizable templates

## Upstream

- Project: <https://github.com/goauthentik/authentik>
- Website: <https://goauthentik.io>
- Container image: `ghcr.io/goauthentik/server:2026.8.5` (server, worker)
- Dependencies: PostgreSQL 18 (`aac-authentik-db`), Redis 8.6.1 (`aac-authentik-redis`)
- License: MIT for the open-source core; the `authentik/enterprise/` directory is licensed separately (upstream project)

## Components

| Service | Container | Image |
|---|---|---|
| Server (web UI, API, SSO endpoints) | `aac-authentik` | `ghcr.io/goauthentik/server:2026.8.5` |
| Worker (background tasks, outposts) | `aac-authentik-worker` | `ghcr.io/goauthentik/server:2026.8.5` |
| Database | `aac-authentik-db` | `postgres:18` |
| Cache / message broker | `aac-authentik-redis` | `redis:8.6.1` |

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. For a
manual run, the external networks used for routing must exist first:

```bash
docker network create services-secured
docker network create services-exposed

# Provide the values listed under "Configuration" in .env / stack.env
docker compose up -d
```

Secrets are not stored in this repository. Provide them through the deployment pipeline
or create the `.env` and `stack.env` files yourself (see below). Do not commit them.

## Configuration

Environment variables (defaults from `service.yml`; `CHANGE_ME` must be set per deployment).
Variables marked *(secret)* are provided through the secrets mechanism and must not be committed.

| Variable | Default | Description |
|---|---|---|
| `AUTHENTIK_SECRET_KEY` | `CHANGE_ME` (secret) | Secret key used by authentik to sign sessions and tokens |
| `AUTHENTIK_POSTGRESQL__HOST` | `CHANGE_ME` | Hostname of the PostgreSQL server (the database service) |
| `AUTHENTIK_POSTGRESQL__USER` | `authentik` | PostgreSQL user for authentik |
| `AUTHENTIK_POSTGRESQL__NAME` | `authentik` | PostgreSQL database name for authentik |
| `AUTHENTIK_POSTGRESQL__PASSWORD` | `CHANGE_ME` (secret) | PostgreSQL password for authentik |
| `AUTHENTIK_REDIS__HOST` | `CHANGE_ME` | Hostname of the Redis server (the redis service) |
| `AUTHENTIK_EMAIL__HOST` | `CHANGE_ME` | SMTP server host |
| `AUTHENTIK_EMAIL__PORT` | `465` | SMTP server port |
| `AUTHENTIK_EMAIL__USERNAME` | `CHANGE_ME` | SMTP username |
| `AUTHENTIK_EMAIL__PASSWORD` | `CHANGE_ME` (secret) | SMTP password |
| `AUTHENTIK_EMAIL__FROM` | `CHANGE_ME` | Sender address for authentik emails |
| `AUTHENTIK_EMAIL__USE_TLS` | `true` | Use STARTTLS for SMTP |
| `AUTHENTIK_EMAIL__USE_SSL` | `false` | Use implicit SSL for SMTP |
| `AUTHENTIK_EMAIL__TIMEOUT` | `10` | SMTP timeout in seconds |
| `AUTHENTIK_ERROR_REPORTING__ENABLED` | `true` | Send error reports to upstream (Sentry) |
| `HOMEPAGE_WIDGET_KEY` | `CHANGE_ME` (secret) | API key for the Homepage dashboard widget |
| `POSTGRES_USER` | `authentik` (secret) | User of the PostgreSQL database service |
| `POSTGRES_DB` | `authentik` (secret) | Database name of the PostgreSQL database service |
| `POSTGRES_PASSWORD` | `CHANGE_ME` (secret) | Password of the PostgreSQL database service |
| `PUID` / `PGID` | `1000` | User and group ID the container runs as |
| `TZ` | `Europe/Berlin` | Time zone |

Ports:

| Port | Protocol | Purpose |
|---|---|---|
| `9000` | TCP | HTTP interface (`web`) |
| `9443` | TCP | HTTPS interface (`websecure`) |

Volumes:

| Mount | Description |
|---|---|
| `media` -> `/media` | Media files (worker and server) |
| `custom-templates` -> `/templates` | Custom templates |
| `certs` -> `/certs` | Certificates (worker) |
| `docker_sock` -> `/var/run/docker.sock` | Host Docker socket (bind mount), used by the worker's Docker integration |
| `redis-data` -> `/data` | Redis data |
| `db` -> `/var/lib/postgresql` | PostgreSQL data (database service) |

The worker sets `nofile` ulimit 65536 and a memory limit of 4 GB (`mem_limit: 4g`).

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-authentik>.

## Credits and license

authentik is developed by Jens Langhammer and the authentik community. The open-source core
is licensed under the MIT license; the `authentik/enterprise/` directory is licensed separately.
This repository only contains the deployment definition for it and is not a fork of the
application. The logo is provided by the authentik project; see
[`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md) for icon sources and licenses.
