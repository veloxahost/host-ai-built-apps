# Host AI-built apps on your own domain

Exported your app from **Lovable, Bolt, v0, Base44, Replit or Veloxa Build** and want it on your own domain, on hosting you control? This repo has the configs to do it in about 15 minutes: static hosting, single-page-app routing, Docker builds, HTTPS and CI deploys.

## 1. Figure out what you exported

| You have… | It is | Use |
|---|---|---|
| `index.html` + `.css`/`.js` files, no `package.json` | Plain static site (Veloxa Build download, many simple exports) | [`caddy/Caddyfile.static`](caddy/Caddyfile.static) or [`nginx/static.conf`](nginx/static.conf) |
| `package.json` with `vite` in it | Vite single-page app (typical Lovable, Bolt export) | Build with `npm run build`, serve `dist/` with [`caddy/Caddyfile.spa`](caddy/Caddyfile.spa) or [`nginx/spa.conf`](nginx/spa.conf) |
| `package.json` with `next` | Next.js app (typical v0 export) | [`docker/Dockerfile.next`](docker/Dockerfile.next) |
| `package.json` with `@tanstack/start` or a `server` entry | Full-stack Node app | [`docker/Dockerfile.node`](docker/Dockerfile.node) |

> **Backend check.** If the app uses the builder's own database, auth or storage (for example Supabase via Lovable, or Veloxa Build's built-in services), that part keeps talking to those services. Test sign-in, forms and uploads on the new domain, and update allowed origins/redirect URLs in that service.

## 2. Static site or Vite SPA on a server (Caddy, automatic HTTPS)

```bash
# On an Ubuntu server with your domain's A record pointing at it
sudo apt install -y caddy
sudo mkdir -p /var/www/app
# copy your files (static) or your dist/ folder (Vite) into /var/www/app
sudo cp caddy/Caddyfile.spa /etc/caddy/Caddyfile   # or Caddyfile.static
sudo sed -i 's/app.example.com/YOUR.DOMAIN/' /etc/caddy/Caddyfile
sudo systemctl reload caddy
```

Caddy gets a Let's Encrypt certificate on first request. The SPA config sends unknown paths to `index.html`, so client-side routes like `/dashboard` don't 404 on refresh — the #1 problem after exporting a Lovable or Bolt app.

## 3. Next.js or Node apps with Docker

```bash
cp docker/Dockerfile.next ./Dockerfile      # or Dockerfile.node
docker build -t myapp .
docker run -d --restart unless-stopped -p 127.0.0.1:3000:3000 --env-file .env myapp
```

Then reverse-proxy from Caddy: [`caddy/Caddyfile.proxy`](caddy/Caddyfile.proxy).

## 4. Deploy on every push (GitHub Actions)

[`github-actions/deploy-static.yml`](github-actions/deploy-static.yml) builds a Vite app (or skips the build for plain static files) and uploads it to your server over SSH with `rsync`. Add three repository secrets: `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_SSH_KEY`.

## Common gotchas

- **Blank page after deploy** → the app was built for a sub-path. Set `base: '/'` in `vite.config` or open it at the right path.
- **Routes 404 on refresh** → use the SPA config (fallback to `index.html`).
- **Env vars missing** → Vite only exposes `VITE_*` variables, and they're baked in at build time. Rebuild after changing them.
- **Auth redirect fails** → add the new domain to your auth provider's allowed redirect URLs.
- **Mixed content warnings** → every API URL must be `https://`.

## Where to host

Any Linux VPS works. On [VeloxaHost](https://veloxahost.com/veloxa-build/guides/where-to-host-an-ai-built-website?utm_source=github&utm_medium=referral&utm_campaign=host-ai-built-apps) you can use managed hosting (Git deploys, free HTTPS) from $6.99/mo or a root server from $4/mo — and [Veloxa Build](https://veloxahost.com/veloxa-build?utm_source=github&utm_medium=referral&utm_campaign=host-ai-built-apps) hosts what it builds on every plan, including Free.

Step-by-step guides:
- [Host a Lovable app on your own domain](https://veloxahost.com/veloxa-build/guides/host-a-lovable-app-on-your-own-domain?utm_source=github&utm_medium=referral&utm_campaign=host-ai-built-apps)
- [Export a Bolt project and host it](https://veloxahost.com/veloxa-build/guides/export-a-bolt-project-and-host-it?utm_source=github&utm_medium=referral&utm_campaign=host-ai-built-apps)

## License

MIT
