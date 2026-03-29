# Headscale + SWAG Docker Setup

This repo is a cleaned template for running:

- `headscale` in Docker
- `swag` as the TLS reverse proxy
- DuckDNS-based DNS validation for Let's Encrypt certificates

Only reusable setup files are kept here. Generated runtime state, logs, certificates, databases, and credentials are excluded.

## What You Configure

- `docker/.env` for container runtime values such as UID, GID, timezone, and base domain
- `docker/headscale/config/config.yaml` for the public Headscale URL and server settings
- `docker/swag/config/dns-conf/duckdns.ini` for the DuckDNS API token
- `docker/swag/config/nginx/proxy-confs/headscale.subdomain.conf` for the public Headscale hostname

## Repo Layout

```text
docker/
  .env.example
  docker-compose.yml
  SETUP.md
  headscale/
    config/
      config.yaml.example
    lib/
      .gitkeep
  swag/
    config/
      dns-conf/
        duckdns.ini.sample
      nginx/
        proxy-confs/
          headscale.subdomain.conf.sample
```

## Quick Start

1. Copy the example files to their runtime names.
2. Replace all placeholders with your own domain and system values.
3. Start `swag`, then start `headscale`.

Detailed steps are in `docker/SETUP.md`.

## Notes

- `docker-compose.yml` is parameterized through `docker/.env`.
- SWAG generates most of its internal config on first start; those generated files are intentionally ignored.
- The repo keeps `docker/headscale/lib/.gitkeep` so the bind-mounted data directory exists without committing the database.
