# AGENTS.md — Base44 dev environment

## What this is
A static single-page site builder. No build step, no dependencies:
- `builder.html` — the builder UI (brief form, live preview, export)
- `tpl-engine.js` — 88 KB template engine loaded by builder.html
- `i18n.js` — translations for the builder UI; fully client-side, nothing to call
- `assistant.js` — the on-page assistant widget
- `media/tpl/` — the twelve template photographs the tiles show
- `manifest.json` and the favicons — the page asked for them by absolute path and got 404s

The shared genvidpro.com chrome (`gv-chrome.js`) is deliberately not loaded here: it depends
on `/push.js`, `/tts`, `/privacy.html`, `/terms.html` and `/work`, which belong to the site
repo, not to the builder.

## How it runs here
Served by `nginx:alpine` via `docker-compose.base44.yml`. The repo is bind-mounted read-only at `/app`; nginx serves it on port 3000.

- `nginx.base44.main.conf` → mounted as `/etc/nginx/nginx.conf` (sets `user root` because the sandbox repo files are mode 700 and nginx's default `nginx` user can't read them).
- `nginx.base44.conf` → mounted as `/etc/nginx/conf.d/default.conf` (serves `builder.html` as the index).

No live-reload dev server (pure static files). After editing, call `reload_preview` so the user sees the change.

`nginx.base44.conf` closes `/AGENTS.md`, `/README.md`, the compose file, its own config and
`.base44/` — the whole repo is the web root here, so they were downloadable. It also answers
404 for a missing file instead of falling back to `builder.html`: the fallback meant a missing
photograph came back as 200 with the entire builder page in the body, which reads as working.

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

## The preview address has to be short enough to resolve

The sandbox hostname is `<port>-<appId>[--b-<short>]-<sandboxId>.imported.base44-preview.app`,
and a DNS label may hold 63 characters. On **main** that label is 54 and resolves. On a
**named branch** Base44 inserts `--b-xxxxxxx`, which takes it to 65 — over the limit, so the
address resolves nowhere, in any browser or client. Measured 30.09.2026:

    3000-<appId>-<sandboxId>                 54 characters, resolves
    3000-<appId>--b-xxxxxxx-<sandboxId>      65 characters, "label too long"

So the preview has to run on **main**. A branch preview of this repo cannot be opened at all,
and the failure looks nothing like the 403 this setup was built to fix: the name simply does
not exist, and nothing ever reaches nginx.
