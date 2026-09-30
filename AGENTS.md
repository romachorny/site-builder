# AGENTS.md — Base44 dev environment

## What this is
A static single-page site builder. No build step, no dependencies:
- `builder.html` — the builder UI (brief form, live preview, export)
- `tpl-engine.js` — 88 KB template engine loaded by builder.html
- `i18n.js` — translations for the builder UI; fully client-side, nothing to call
- `assistant.js` — the on-page assistant widget

The shared genvidpro.com chrome (`gv-chrome.js`) is deliberately not loaded here: it depends
on `/push.js`, `/tts`, `/privacy.html`, `/terms.html` and `/work`, which belong to the site
repo, not to the builder.

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

## What does not work without a backend
The builder itself is fully client-side. Two things on the page reach for a server that
does not exist in this repo:

- `assistant.js` posts to `/chat`. On genvidpro.com that is a Cloudflare Function; here it
  answers 404, so the assistant renders but cannot reply. A Base44 backend function would
  fill this in — that feature starts on the Builder plan.
- The generated sites load their photographs from `https://genvidpro.com/media/photo/` and
  `https://genvidpro.com/videos/tpl/` by absolute address. That is on purpose, not a bug:
  the visitor downloads one HTML file and opens it from disk, where a relative path would
  not resolve.

## Secrets
None. Nothing in this repo needs a key, and nothing should be committed with one.
