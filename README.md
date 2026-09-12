# sunnahsky-infra

The live Caddy config for `sunnahsky.com` / `app.sunnahsky.com`, tracked in git instead of only existing as a hand-edited file on the droplet. Started after several same-night edits to `/pds/caddy/etc/caddy/Caddyfile` (the `/tls-check` fix, the `.guest.` wildcard fix, the `app.sunnahsky.com` on-demand switch, the apex root redirect) left no diffable history - only timestamped backup copies and prose descriptions in `HANDOFF.md`.

**Not yet deployed:** the `Caddyfile` in this repo's `sunnahsky-hostname-move` branch reflects the target state of "PDS hostname move and public URL scheme" - `sunnahsky.com` becomes the web app, `pds.sunnahsky.com` becomes the PDS's own host, `app.sunnahsky.com`/`www.sunnahsky.com` become permanent 301s, and `*.sunnahsky.com`/`*.guest.sunnahsky.com` redirect a handle to its canonical profile path (proxying only `/.well-known/atproto-did`). The live droplet still runs the pre-move config on `main`. See the plan file for the full sequencing - this is a two-step deploy (the hostname move, then the handle-redirect split), not one, and needs the PDS's own hostname/env changes live first.

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
