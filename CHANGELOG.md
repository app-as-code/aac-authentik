# Changelog

All notable changes to this deployment definition are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Changed

- Worker: `nofile` ulimit raised to 65536 and memory limit set to 4 GB to contain file descriptor leaks.

## [0.0.1]

### Added

- Initial App-as-Code service definition for authentik with PostgreSQL 18 and Redis 8.6.1 dependencies.
- Server and worker containers running `ghcr.io/goauthentik/server:2026.8.5`.
- HTTP and HTTPS ports with Traefik routing and automatic DNS.
- README with configuration reference and project icon.
