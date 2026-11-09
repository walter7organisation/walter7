# walter7.lol/ping — deploy steps

## 1. Website repo (GitHub Pages)

Three files, commit and push from your machine (or anywhere):

- `ping/index.html` — the live ping page (fetches stats from `https://ping.walter7.lol`)
- `ping/graph.html` — NEW interactive historical graphs (fetches `/api/series` from the same API)
- `index.html` — homepage with the "proxy ping" banner card linking to `/ping/`

```
git clone git@github.com:walter7organisation/walter7.git
cd walter7
cp /path/to/bundle/repo/index.html .
mkdir -p ping && cp /path/to/bundle/repo/ping/*.html ping/
git add index.html ping/
git commit -m "ping page: add interactive graphs"
git push
```

GitHub Pages deploys in ~1 min. Then open https://walter7.lol/ping/graph.html —
the live page footer also has a "graphs" link.

## 2. Vultr API (already done — only needed if rebuilding the box)

The `/api/series` endpoint (added for the graphs) is live on the `ping-web`
systemd user unit. Verify:

```
curl -s 'https://ping.walter7.lol/api/series?h=24' | head -c 300
```

If missing (fresh box): copy `web.py` from the bundle to
`/home/hermes/services/ping-web/web.py` and
`systemctl --user restart ping-web` as hermes.

## 3. Verify

- `curl -s https://ping.walter7.lol/api/stats | head -c 200`  (JSON, CORS header)
- open https://walter7.lol/ping/ — cards updating every 10s, "graphs" in the footer
- open https://walter7.lol/ping/graph.html — chart with range buttons
  (1h/6h/24h/48h/7d), leg toggles, linear/log scale, spike markers (orange dots)

## Notes

- The API service reads the SLP SQLite DBs read-only per request; nothing to
  restart unless it breaks: `systemctl --user restart ping-web` as hermes.
- Metric = pong_ms (network RTT), same as the Discord stats bot.
- Graph page times render in the visitor's local timezone.
- `?api=` query param overrides API_BASE (used for local testing).
