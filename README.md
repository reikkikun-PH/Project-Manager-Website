# Project Manager — Stable Link (GitHub Pages)

The manager's tunnel URL is **random every launch** (`*.trycloudflare.com`).
This folder is a **static entry page that never moves**. Share *its* address,
not the tunnel address.

## How it works

```
host.py  →  logs/tunnel-url.txt (random, new each run)
              │  publish-live.py --watch
              ▼
Project Manager Website/live.json  →  git push  →  GitHub Pages rebuilds (~30-60s)
              │
index.html (this page) fetches live.json, shows LAUNCH + per-project links,
             optionally auto-redirects with ?go=3
```

- `index.html` — just a loading screen. Reads `live.json`, then verifies
  the address through the server's own `/api/session` (CORS-open JSON
  `{ok, instance}`) and forwards **only on a real answer with the matching
  instance** — a dead tunnel's Cloudflare error page is HTML with no CORS
  headers, so the browser itself rejects it and no redirect happens. If the
  server stays unreachable ~12s it honestly says so ("SERVER OFFLINE",
  last-online time, retry button) and still jumps on its own when `host.py`
  comes back. Nothing else is shown.
- `live.json` — overwritten by the publisher on every launch. Do not hand-edit.
- `tunnel-url.txt` — plain-text copy of the same URL (fallback + `curl` friendly).

## Setup (once)

1. Create a **public** repo, e.g. `Project-Manager-Live`, and push this folder's
   content to its root (or keep it as `docs/` / a subfolder — just note the URL).
2. Repo → **Settings → Pages** → Deploy from branch → `main` + `/ (root)` → Save.
   Your stable link is `https://<user>.github.io/<repo>/`.
3. On the host PC, run the publisher alongside the manager (see below).
4. Share the stable link — visitors see only a loading spinner, then land
   on the live server.

## Running the publisher

From the **Server Project Manager** folder:

```bat
REM one-off update (after host.py prints the PUBLIC LINK):
py -3 publish-live.py --once

REM or keep it watching (leave running next to host.py):
py -3 publish-live.py --watch
```

It reads `logs/tunnel-url.txt` (+ project list from `store.py`),
writes `live.json` + `tunnel-url.txt` here, and — if this folder is a git
checkout — commits and pushes automatically.

Options: `--server-dir`, `--site-dir`, `--instance`, `--no-push`, `--interval`.

## Offline behaviour

When the host PC is off, `live.json` keeps the *last* URL but the page shows
**OFFLINE** and disables Launch (the tunnel is dead anyway — `host.py` deletes
`tunnel-url.txt` on exit and the publisher propagates that).

## Want no sync step at all?

Use a Cloudflare **named tunnel** with your own domain instead of a quick
tunnel — the address never changes, so this page becomes unnecessary:

```sh
cloudflared tunnel login
cloudflared tunnel create pm
cloudflared tunnel route dns pm pm.yourdomain.com
cloudflared tunnel run --url http://127.0.0.1:8000 pm
```

Then share `https://pm.yourdomain.com` directly.
