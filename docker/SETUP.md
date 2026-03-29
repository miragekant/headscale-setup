Setup order for this stack:

1. Create runtime config files from the examples
   - Copy `.env.example` to `.env`
   - Copy `./headscale/config/config.yaml.example` to `./headscale/config/config.yaml`
   - Copy `./swag/config/dns-conf/duckdns.ini.sample` to `./swag/config/dns-conf/duckdns.ini`
   - Copy `./swag/config/nginx/proxy-confs/headscale.subdomain.conf.sample` to `./swag/config/nginx/proxy-confs/headscale.subdomain.conf`

2. Fill in placeholders
   - Replace all `YOUR_*` values in `.env`
   - Set `BASE_DOMAIN` in `.env` to your DuckDNS root domain such as `example.duckdns.org`
   - Replace `HEADSCALE_FQDN` in `./headscale/config/config.yaml`
   - Replace `HEADSCALE_FQDN` in `./swag/config/nginx/proxy-confs/headscale.subdomain.conf`
   - Replace `YOUR_DUCKDNS_TOKEN` in `./swag/config/dns-conf/duckdns.ini`
   - Typical Headscale hostname pattern: `hs.<your-base-domain>`

3. Protect the DNS credential file
   - Run: `chmod 600 ./swag/config/dns-conf/duckdns.ini`

4. Start the reverse proxy first
   - Run: `docker compose up -d swag`
   - Check logs: `docker logs -f swag`
   - SWAG will generate the rest of its working config under `./swag/config`

5. Start Headscale
   - Run: `docker compose up -d headscale`
   - Check logs: `docker logs -f headscale`

6. Verify
   - Open `https://HEADSCALE_FQDN`
   - Confirm services are healthy:
     - `docker compose ps`
     - `docker exec headscale headscale health`
      - `docker logs --tail 100 swag`
      - `docker logs --tail 100 headscale`
