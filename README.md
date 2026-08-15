# sunnahsky-infra

The live Caddy config for `sunnahsky.com` / `app.sunnahsky.com`, tracked in git instead of only existing as a hand-edited file on the droplet. Started after several same-night edits to `/pds/caddy/etc/caddy/Caddyfile` (the `/tls-check` fix, the `.guest.` wildcard fix, the `app.sunnahsky.com` on-demand switch, the apex root redirect) left no diffable history - only timestamped backup copies and prose descriptions in `HANDOFF.md`.

## Layout

- `Caddyfile` - the actual served config.
- `/pds/caddy/etc/caddy/` on the droplet **is** a checkout of this repo (not a copy synced into it). `sunnahsky-web/` (the deployed static web build) lives alongside the Caddyfile in that same directory but is gitignored here - it's deployed separately, via `social-app`'s web export + `rsync`.

## Deploying a Caddyfile change

From the droplet:

```bash
cd /pds/caddy/etc/caddy
git pull
docker exec caddy caddy validate --config /etc/caddy/Caddyfile --adapter caddyfile
docker exec caddy caddy reload --config /etc/caddy/Caddyfile --adapter caddyfile
```

Always validate before reload. Confirm production afterward (health endpoint, an existing account handle, and - if the change touches TLS/on-demand config specifically - a deliberate `docker restart caddy` to prove it survives a cold certificate cache, not just a hot reload).

Don't hand-edit the live file directly - that's exactly the drift this repo exists to stop.
