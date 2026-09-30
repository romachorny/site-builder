# AGENTS.md — Base44 dev environment

## What this is
A static single-page site builder. Two files, no build step, no backend, no dependencies:
- `builder.html` — the builder UI (brief form, live preview, export)
- `tpl-engine.js` — 88 KB template engine loaded by builder.html

## How it runs here
Served by `nginx:alpine` via `docker-compose.base44.yml`. The repo is bind-mounted read-only at `/app`; nginx serves it on port 3000.

- `nginx.base44.main.conf` → mounted as `/etc/nginx/nginx.conf` (sets `user root` because the sandbox repo files are mode 700 and nginx's default `nginx` user can't read them).
- `nginx.base44.conf` → mounted as `/etc/nginx/conf.d/default.conf` (serves `builder.html` as the index).

No live-reload dev server (pure static files). After editing, call `reload_preview` so the user sees the change.

## Verifying it works
```sh
docker compose -f docker-compose.base44.yml ps          # should show "healthy"
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/   # 200
```

## Secrets
None needed — fully client-side, no external services.
