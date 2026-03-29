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

## Setup

1. Copy the example files to their runtime names.
   - `cp docker/.env.example docker/.env`
   - `cp docker/headscale/config/config.yaml.example docker/headscale/config/config.yaml`
   - `cp docker/swag/config/dns-conf/duckdns.ini.sample docker/swag/config/dns-conf/duckdns.ini`
   - `cp docker/swag/config/nginx/proxy-confs/headscale.subdomain.conf.sample docker/swag/config/nginx/proxy-confs/headscale.subdomain.conf`
2. Replace all placeholders with your own domain and system values.
   - Set `BASE_DOMAIN` in `docker/.env` to your DuckDNS root domain such as `example.duckdns.org`
   - Replace `HEADSCALE_FQDN` in `docker/headscale/config/config.yaml`
   - Replace `HEADSCALE_FQDN` in `docker/swag/config/nginx/proxy-confs/headscale.subdomain.conf`
   - Replace `YOUR_DUCKDNS_TOKEN` in `docker/swag/config/dns-conf/duckdns.ini`
   - A typical Headscale hostname is `hs.<your-base-domain>`
3. Protect the DNS credential file.
   - `chmod 600 docker/swag/config/dns-conf/duckdns.ini`
4. Start `swag` first.
   - `cd docker`
   - `docker compose up -d swag`
   - `docker logs -f swag`
5. After adding the DNS token, restart `swag`.
   - `docker compose restart swag`
6. Confirm certificate issuance.
   - Look for log lines such as `Successfully received certificate.` and `Server ready`
7. Start `headscale`.
   - `docker compose up -d headscale`
   - `docker logs -f headscale`
8. Verify the stack.
   - `docker compose ps`
   - `docker exec headscale headscale health`
   - `docker logs --tail 100 swag`
   - `docker logs --tail 100 headscale`
   - Open `https://HEADSCALE_FQDN`
   - A successful public check should return HTTP 200

## Operational Notes

- `docker-compose.yml` is parameterized through `docker/.env`.
- SWAG generates most of its internal config on first start; those generated files are intentionally ignored.
- The repo keeps `docker/headscale/lib/.gitkeep` so the bind-mounted data directory exists without committing the database.
- If Headscale logs `Listening without TLS but ServerURL does not start with http://`, that is expected when TLS is terminated by SWAG.
- The SWAG proxy example assumes the upstream service name is `headscale` on the Docker network.
- Persistent directories should exist before first startup:
  - `docker/headscale/config`
  - `docker/headscale/lib`
  - `docker/swag/config`

## Admin Commands

```bash
cd docker
docker compose ps
docker logs --tail 100 swag
docker logs --tail 100 headscale
docker exec headscale headscale health
docker exec headscale headscale users list --output json
docker exec headscale headscale nodes list --output json
```

## Enrollment Examples

Create a user:

```bash
docker exec headscale headscale users create USERNAME
```

Create a reusable preauth key for numeric user ID `1`:

```bash
docker exec headscale headscale preauthkeys create --user 1 --reusable --expiration 24h --output json
```

Register a mobile device manually when Headscale provides a registration key:

```bash
docker exec headscale headscale nodes register --user USERNAME --key <REGISTRATION_KEY>
```

## WSL Notes

- If your WSL environment does not run `systemd`, `sudo systemctl start tailscaled` will not work.
- A working alternative is `sudo tailscaled --tun=userspace-networking`.
- Then connect with `sudo tailscale up --login-server https://HEADSCALE_FQDN --authkey <AUTH_KEY>`.
- In this mode, manual Taildrop retrieval is reliable:
  - `tailscale file get ~/Downloads`

## Exit Node Clarification

- `headscale` is the control plane.
- An exit node is a separate Tailscale client that forwards internet traffic.

You do not need an exit node just to connect devices to each other through Headscale.
