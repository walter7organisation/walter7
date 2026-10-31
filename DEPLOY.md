# walter7.lol/ping — deploy steps

## 1. Website repo (GitHub Pages)

Two changed files, commit and push from your machine (or anywhere):

- `index.html` — homepage with a new "proxy ping" banner card linking to `/ping/`
- `ping/index.html` — the live ping page (fetches stats from `https://ping.walter7.lol`)

```
git clone git@github.com:walter7organisation/walter7.git
cd walter7
cp /path/to/bundle/repo/index.html .
mkdir -p ping && cp /path/to/bundle/repo/ping/index.html ping/
git add index.html ping/
git commit -m "add live proxy ping page"
git push
```

GitHub Pages deploys in ~1 min. Page will show "api unreachable" until step 3 is done.

## 2. Vultr apache vhost (needs sudo, as josie)

```
sudo cp /home/hermes/services/ping-web/ping.walter7.lol.conf /etc/apache2/sites-available/
sudo a2ensite ping.walter7.lol
sudo apachectl configtest
sudo systemctl reload apache2
```

The vhost proxies `/api` to the local service (port 8091) and redirects
everything else to https://walter7.lol/ping/.

## 3. Cloudflare DNS

Add an A record: `ping` -> `173.199.93.195` (proxied, same settings as `uploads`).

## 4. Verify

- `curl -s https://ping.walter7.lol/api/stats | head -c 200`  (JSON, CORS header)
- open https://walter7.lol/ping/ — three cards updating every 10s

## Notes

- The stats service itself is already running on Vultr as the `ping-web`
  systemd user unit (hermes), reading the two SLP SQLite DBs. Nothing to do
  there unless it needs a restart: `systemctl --user restart ping-web` as
  hermes, or ask the agent.
- Metric = pong_ms (network RTT), same as the Discord stats bot.
