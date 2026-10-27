# Deploying

`index.html` is a single static file with no build step, so serving it is just a
web server pointed at this repo.

## On a Linux + Docker host

```sh
git clone https://github.com/trgayd/Scotland-Distilleries.git ~/projects/Scotland-Distilleries
cd ~/projects/Scotland-Distilleries/deploy
cp .env.example .env     # adjust BIND_ADDR and PORT if needed
docker compose up -d
```

Defaults to `127.0.0.1:8080`, so nothing is exposed beyond the machine until you
say so. Works under rootless Docker — no privileged ports.

To update, `git pull` and reload the page: `index.html` is bind-mounted, so no
rebuild or restart is needed.

### Exposing it

- **LAN** — set `BIND_ADDR` to the machine's LAN IPv4 in `.env`, then
  `docker compose up -d`. Check the host firewall allows the port.
- **Tailnet over HTTPS** — leave it on `127.0.0.1` and put Tailscale Serve in
  front:

  ```sh
  tailscale serve --bg --https=10000 http://127.0.0.1:8080
  ```

  Serve only allows HTTPS on ports 443, 8443 and 10000.

## Elsewhere

Any static host works, with no change to the file: GitHub Pages, S3 + CloudFront,
AWS Amplify, Netlify. The page loads React, Babel and Tailwind from public CDNs
at runtime, so the host needs no build tooling — but it does mean the page needs
internet access to render, and will not work on an air-gapped network.

## Note on state

Marks (Liked / Visited / To visit) are stored in the browser's `localStorage`.
They are per browser and per device and never reach the server, so they are not
captured by any backup of the host, and serving the same file from a different
origin (a different hostname, port or scheme) will not carry them across.
