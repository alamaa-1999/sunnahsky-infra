# sunnahsky-infra

The live Caddy config for `sunnahsky.com` / `app.sunnahsky.com`, tracked in git instead of only existing as a hand-edited file on the droplet. Started after several same-night edits to `/pds/caddy/etc/caddy/Caddyfile` (the `/tls-check` fix, the `.guest.` wildcard fix, the `app.sunnahsky.com` on-demand switch, the apex root redirect) left no diffable history - only timestamped backup copies and prose descriptions in `HANDOFF.md`.

**Stage 1 of "PDS hostname move and public URL scheme" is live** (deployed 2026-09-14, `main` at `c523e18`): `sunnahsky.com` is the web app, `pds.sunnahsky.com` is the PDS's own host, `app.sunnahsky.com` is a permanent 301. Wildcard handle hosts (`*.sunnahsky.com`/`*.guest.sunnahsky.com`) still plain-proxy every path exactly as before the move - the path-based canonical-profile redirect is Stage 2, not yet deployed. See HANDOFF.md's 2026-09-14/17 entries for the full incident record: the first version of this stage broke TLS on `app.`/`pds.sunnahsky.com` for three days by switching them off `tls { on_demand }`, since fixed. `www.sunnahsky.com`'s block exists but doesn't work yet - blocked on a PDS code fix (`ADDITIONAL_APPROVED_DOMAINS`), not a Caddy change. See the plan file for Stage 2's own sequencing.

**Every explicit host block in this Caddyfile uses `tls { on_demand }` except the apex (`sunnahsky.com`), which deliberately does not** - see that block's own comment before changing this. This asymmetry is load-bearing, confirmed by the incident above, not an oversight to "clean up."

**Phase 4a-1 (head service) - live since 2026-09-19 (`main` at `dde939f`).** The apex block splits into an asset matcher (file_server) and a catch-all that proxies navigations to the `head` container on `localhost:3010`, with `handle_errors` falling back to the plain shell when the service is down. `head.env.example` and `compose.head.example.yaml` are the two droplet-side files it needs (`/pds/head.env`, and a fragment for `/pds/compose.yaml`). The asset matcher lists the web export's top-level files by name; a web export that adds a new top-level file needs the matcher updated in the same deploy. Post-deploy smoke test: `scripts/head-smoke.sh` in the workspace (a copy lives at `/pds/head-smoke.sh` on the droplet); run it against the public URL after any change to the apex block. Note `/static/style.css` is a genuine 404: upstream's `index.html` template links a stylesheet the web export never emits, and the old `try_files` fallback used to mask that by serving the shell.

## Layout

- `Caddyfile` - the actual served config.
- `head.env.example` - the head service's env file, copied to `/pds/head.env` (no secrets).
- `compose.head.example.yaml` - the `head` service fragment for `/pds/compose.yaml`.
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
