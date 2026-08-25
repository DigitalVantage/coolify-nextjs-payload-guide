# Coolify Deployment Guide: Next.js + Payload CMS + MongoDB

> Self-host a production **Next.js** + **Payload CMS** + **MongoDB** stack on a single VPS using [**Coolify**](https://coolify.io). Step-by-step, copy-paste-ready, written from a real production deployment.
>
> Maintained by **[Digital Vantage](https://www.digitalvantage.pl)**. Pairs with the [`nextjs-payload-starter`](https://github.com/DigitalVantage/nextjs-payload-starter) repo, which ships with a Coolify-ready Dockerfile.

[![Coolify](https://img.shields.io/badge/Coolify-self--hosted-6B46C1)](https://coolify.io)
[![Next.js](https://img.shields.io/badge/Next.js-16-black?logo=next.js)](https://nextjs.org/)
[![Payload](https://img.shields.io/badge/Payload-3-000000?logo=payloadcms)](https://payloadcms.com/)
[![MongoDB](https://img.shields.io/badge/MongoDB-7-47A248?logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Table of contents

- [What you'll have at the end](#what-youll-have-at-the-end)
- [Why self-host with Coolify](#why-self-host-with-coolify)
- [Prerequisites](#prerequisites)
- [Step 1 — Install Coolify on the VPS](#step-1--install-coolify-on-the-vps)
- [Step 2 — Point a domain at the VPS via Cloudflare](#step-2--point-a-domain-at-the-vps-via-cloudflare)
- [Step 3 — Add Coolify's own domain (and SSL)](#step-3--add-coolifys-own-domain-and-ssl)
- [Step 4 — Create the project + add MongoDB](#step-4--create-the-project--add-mongodb)
- [Step 5 — Connect your GitHub repo](#step-5--connect-your-github-repo)
- [Step 6 — Environment variables](#step-6--environment-variables)
- [Step 7 — Persistent volume for media uploads](#step-7--persistent-volume-for-media-uploads)
- [Step 8 — Deploy](#step-8--deploy)
- [Step 9 — Flip Cloudflare to proxied + verify SSL](#step-9--flip-cloudflare-to-proxied--verify-ssl)
- [Step 10 — Auto-deploy on git push](#step-10--auto-deploy-on-git-push)
- [Database backups](#database-backups)
- [Troubleshooting](#troubleshooting)
- [Security checklist](#security-checklist)
- [Performance tips](#performance-tips)
- [Going further](#going-further)
- [Useful links](#useful-links)

## What you'll have at the end

After ~45 minutes of focused work:

- **Next.js 16 + Payload CMS 3** app running on your own VPS, served at your domain over HTTPS.
- **MongoDB 7** running as a managed Coolify service on the same VPS.
- **Automatic redeploys** on every `git push` to `main`.
- **Let's Encrypt SSL** behind a Cloudflare proxy.
- **Persistent media uploads** that survive redeploys.
- **Daily database backups** via Coolify's built-in scheduler.

This guide assumes you start from zero. If you already have Coolify installed, jump to [Step 4](#step-4--create-the-project--add-mongodb).

## Why self-host with Coolify

Three reasons most B2B teams pick this path over Vercel + MongoDB Atlas:

| Concern               | Vercel + Atlas                                     | Coolify on VPS                                                  |
| --------------------- | -------------------------------------------------- | --------------------------------------------------------------- |
| Monthly cost (small)  | $20–50 (Vercel Pro) + $9–57 (Atlas M10) = **$30–100+** | **$5–20** (VPS, all-in)                                         |
| Data sovereignty      | US-based providers, GDPR via SCC                   | Choose your jurisdiction (OVH FR/PL, Hetzner DE, …)             |
| Egress / bandwidth    | Vercel meters bandwidth, Atlas charges transfer    | Most VPS plans include generous transfer (often unmetered)      |
| Lock-in               | Vendor-specific routing, ISR semantics             | Plain Docker containers — move to any other host in an evening  |

**Trade-off:** you own backups, security patching and uptime. If that's not a fair trade, stick with Vercel — there's no shame in not running infra.

This guide is for the case where the trade is worth it: a Polish or EU B2B site that needs predictable costs, data inside the EU, and the freedom to run any side service (queue worker, custom cron, scraper) on the same box.

## Prerequisites

Before starting you need:

1. **A VPS** (any provider). Recommended minimum: **2 vCPU, 4 GB RAM, 40 GB SSD**, Ubuntu 22.04+ or Debian 12+.
   - Tested with: OVH VPS Essential / Value, Hetzner CX22, Mikrus 3.0 Pro.
   - Coolify itself uses ~1 GB RAM at idle; the rest is for your app + MongoDB. Builds need a transient ~2 GB extra (or swap — see [Performance tips](#performance-tips)).
2. **A domain name** with DNS managed in **Cloudflare** (free plan is enough). If your domain is on a different registrar, point its nameservers at Cloudflare first — takes 5 minutes.
3. **A GitHub repository** containing your Next.js + Payload app.
   - If you don't have one yet, fork [`DigitalVantage/nextjs-payload-starter`](https://github.com/DigitalVantage/nextjs-payload-starter) — this guide assumes its layout (`Dockerfile` at the root, Payload's `db-mongodb` adapter, `output: 'standalone'` in `next.config.ts`).
4. **SSH access** to the VPS as `root` or as a sudoer. Key-based auth strongly preferred — see [Security checklist](#security-checklist).

## Step 1 — Install Coolify on the VPS

SSH into your VPS:

```bash
ssh root@<your-vps-ip>
```

Run the official installer:

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

What this does:

- Detects and installs Docker if missing.
- Pulls the `coollabsio/coolify` images and starts the dashboard on port `8000`.
- Sets up the internal Docker network and **Traefik** reverse proxy on ports `80` and `443`.
- Generates an initial admin invite URL printed at the end.

When it finishes, the script prints something like:

```text
Your initial setup URL: http://<vps-ip>:8000
```

Open that in a browser and create the first admin account. Use a strong password — this is the keys-to-the-kingdom UI.

<!-- screenshot 01: Coolify first-run "Register / first admin signup" page. See docs/images/README.md for capture details. -->

> **Pitfall #1:** if your host blocks outgoing connections by default (rare on OVH/Hetzner, common on locked-down corporate networks), the installer hangs on the Docker pull. From a different terminal, SSH in and run `docker ps` — if it returns nothing meaningful for >2 minutes, your firewall is the issue. Allow outbound `:443` and retry.

## Step 2 — Point a domain at the VPS via Cloudflare

In Cloudflare DNS for your domain (`example.com`), create two `A` records:

| Type | Name                | Content       | Proxy status              |
| ---- | ------------------- | ------------- | ------------------------- |
| A    | `coolify`           | `<vps-ip>`    | **DNS only** (gray cloud) |
| A    | `cms` (or `www`, …) | `<vps-ip>`    | **DNS only** for now      |

- `coolify.example.com` is the Coolify dashboard itself. Keep it **DNS-only** permanently — the dashboard doesn't need DDoS protection and Cloudflare proxying interferes with the Let's Encrypt HTTP-01 challenge.
- `cms.example.com` is your actual app. Keep it **DNS-only** *until the first SSL cert is issued*, then flip to **Proxied** in [Step 9](#step-9--flip-cloudflare-to-proxied--verify-ssl).

<!-- screenshot 02: Cloudflare DNS records page with both A records showing gray cloud (DNS only). VPS IP masked. -->

Wait ~30 seconds for DNS to propagate, then verify:

```bash
dig +short coolify.example.com
dig +short cms.example.com
# Both should return your VPS IP
```

> **Pitfall #2:** Cloudflare's "Proxied" mode + Let's Encrypt's HTTP-01 challenge will **conflict on first issuance** — the challenge request lands on Cloudflare's edge, not your VPS. Use the gray cloud until the cert is issued, then flip to orange. Alternative: switch Coolify to the DNS-01 challenge (more setup but works with proxy from day one).

## Step 3 — Add Coolify's own domain (and SSL)

Back in the Coolify UI:

1. **Settings → Configuration → Instance Domain**: set to `https://coolify.example.com`.
2. Click **Save**. Coolify auto-generates a Let's Encrypt cert via Traefik.
3. Wait ~30 seconds, then reload — the URL bar should now show HTTPS for the dashboard itself.

<!-- screenshot 03: Coolify Settings → Configuration with Instance Domain set to https://coolify.example.com. -->

If the cert doesn't appear, check **Servers → localhost → Proxy Logs** for Traefik errors. The most common cause is the DNS A record pointing somewhere else (typo, propagation lag).

## Step 4 — Create the project + add MongoDB

In Coolify:

1. **Projects → + New Project**, name it e.g. `nextjs-payload-prod`.
2. Pick or create a **Server**. The local server is auto-created on install — that's the one you want for a single-VPS setup.
3. Inside the project, click **+ New Resource → Database → MongoDB**.
4. Pick **MongoDB 7**. Set:
   - **Username**: `payload`
   - **Password**: click "Generate" (Coolify produces a strong random one — copy it now or you'll have to look it up later)
   - **Database name**: `payload`
5. Click **Deploy**. After ~30s the database is up.

<!-- screenshot 04: Coolify "+ New Resource → Database → MongoDB" form filled in: username `payload`, db name `payload`, password masked. -->

Coolify shows a **Connection String** in the resource view, looking like:

<!-- screenshot 05: MongoDB resource view with the Connection String field visible. Mask password and container hash. -->

```text
mongodb://payload:<password>@<container-name>:27017/payload?authSource=admin
```

Copy this — you'll paste it as `DATABASE_URL` in [Step 6](#step-6--environment-variables).

> **Note:** the host part is the Docker container name. Coolify auto-resolves it for any other resource in the same project — no need to expose port `27017` externally. **Don't expose `27017` to the public** unless you have a real reason to (and even then, use TLS + IP allow-listing).

## Step 5 — Connect your GitHub repo

1. **Coolify → Sources → + Add → GitHub App**. Install the Coolify GitHub App on your account or organization and grant it access to the repo. This sets up the webhook and OAuth flow in one step.

   <!-- screenshot 06: GitHub "Install Coolify GitHub App" permission screen with the DigitalVantage org row visible. -->

2. Inside the project, click **+ New Resource → Application**.
3. Pick the **GitHub source** you just added, choose your repo, branch `main`.
4. **Build Pack: Dockerfile** (the starter ships one at the project root).
5. Set **Domain** to `https://cms.example.com`.
6. Set **Port Exposes** to `3000` (Next.js standalone server).
7. **Don't deploy yet** — set environment variables first.

<!-- screenshot 07: Coolify application creation form filled in: Build Pack Dockerfile, Domain https://cms.example.com, Port 3000, Branch main. -->

## Step 6 — Environment variables

In the application's **Environment Variables** tab, add:

```env
DATABASE_URL=mongodb://payload:<password>@<mongo-container>:27017/payload?authSource=admin
PAYLOAD_SECRET=<output of: openssl rand -base64 32>
NEXT_PUBLIC_SERVER_URL=https://cms.example.com
CRON_SECRET=<output of: openssl rand -hex 32>
PREVIEW_SECRET=<output of: openssl rand -hex 32>
NODE_ENV=production
```

Generate the secrets locally on your laptop:

```bash
openssl rand -base64 32  # PAYLOAD_SECRET — used to sign JWTs
openssl rand -hex 32     # CRON_SECRET — used by Payload's job runner
openssl rand -hex 32     # PREVIEW_SECRET — validates draft preview URLs
```

Mark `PAYLOAD_SECRET`, `CRON_SECRET`, `PREVIEW_SECRET` as **Secret** in Coolify (the lock icon) so they're hidden from build logs and the UI's edit view.

> **Pitfall #3:** `NEXT_PUBLIC_*` vars are baked into the client JS bundle **at build time**, not runtime. Changing `NEXT_PUBLIC_SERVER_URL` requires a **rebuild**, not just a restart. Coolify shows a "Build Variable" vs "Runtime Variable" toggle next to each env var — set `NEXT_PUBLIC_*` ones to "Build Variable" so they're available during `next build`. Everything else stays as a runtime variable.

<!-- screenshot 08: Coolify Environment Variables tab with all six vars added, Build/Runtime toggle visible, Secret lock icon on at least one secret. Values masked. -->

## Step 7 — Persistent volume for media uploads

Payload writes uploaded images to `public/media/` by default. Without a persistent volume, every redeploy wipes the directory.

In the application config → **Storages** → **+ Add Persistent Storage**:

- **Mount Path**: `/app/public/media`
- **Type**: **Volume** (not bind mount — bind mounts tie you to a specific host path)
- **Volume Name**: `payload-media` (Coolify creates and persists it under `/var/lib/docker/volumes/`)

> **Note:** for production at any meaningful scale, swap the local volume for **S3-compatible object storage** (Hetzner Object Storage, Cloudflare R2, MinIO on a separate VPS, AWS S3). The local-volume approach is fine up to a few GB of uploads — beyond that, restoring from backup gets painful. See [Going further](#going-further).

<!-- screenshot 09: Coolify Persistent Storage form filled in: Mount Path /app/public/media, Type Volume, Volume Name payload-media. -->

## Step 8 — Deploy

Click **Deploy**. Watch the logs in real time:

1. **Pulling repository** — Coolify clones your branch from GitHub.
2. **Building Dockerfile** — runs `pnpm install --frozen-lockfile`, then `pnpm build`.
3. **Starting container** — `node server.js` (Next.js standalone entry point).
4. **Health check passes** — Traefik adds the route and starts serving traffic at `cms.example.com`.

The first build takes **5–10 minutes** with no Docker layer cache. Subsequent builds reuse layers and finish in **1–2 minutes**.

Once the deploy is green, hit `https://cms.example.com/admin` and create the first Payload admin user. Don't lose this password — there's no "forgot password" flow without an email adapter configured (see [Going further](#going-further)).

<!-- screenshot 10: Coolify deploy logs in success state — green status bar, last lines showing "Container started", "Health check passed", route added by Traefik. -->

## Step 9 — Flip Cloudflare to proxied + verify SSL

Once `https://cms.example.com` works with the gray cloud (DNS-only), switch the Cloudflare A record proxy status to **Proxied** (orange cloud). You now get:

- DDoS protection and bot mitigation at Cloudflare's edge.
- Free unmetered bandwidth (egress through Cloudflare doesn't count against your VPS plan).
- HTTP/3, Brotli compression, automatic IPv6.
- Edge caching for `/_next/static/*` (configurable via Cloudflare Page Rules or Cache Rules).

<!-- screenshot 12 (nice-to-have): Cloudflare DNS table after the flip: cms.example.com now has the orange cloud (Proxied). Side-by-side with screenshot 02 ideal but not required. -->

In Cloudflare → **SSL/TLS → Overview**, set the encryption mode to **Full (strict)**. This tells Cloudflare to validate the origin cert (the Let's Encrypt one Coolify just issued) — anything weaker leaves you vulnerable to MITM between Cloudflare and your VPS.

<!-- screenshot 11 (nice-to-have): Cloudflare SSL/TLS Overview with encryption mode Full (strict) highlighted. -->

Verify end-to-end:

```bash
curl -sI https://cms.example.com | head -5
# Expect: HTTP/2 200, server: cloudflare
```

## Step 10 — Auto-deploy on git push

Coolify already configured a webhook when you connected the repo in [Step 5](#step-5--connect-your-github-repo). Test it:

```bash
git commit --allow-empty -m "test deploy"
git push origin main
```

Within 5–10 seconds the Coolify dashboard shows a new deployment running. The webhook delivery and logs are visible in **GitHub → repo → Settings → Webhooks → Recent Deliveries** if you ever need to debug.

<!-- screenshot 13 (nice-to-have): Coolify application Webhooks tab (or GitHub → Settings → Webhooks → Recent Deliveries) showing the auto-deploy webhook firing. -->

## Database backups

Coolify has built-in scheduled backups. In the MongoDB resource → **Backups** tab:

- **Frequency**: `Daily, 02:00` server time (cron: `0 2 * * *`)
- **Retention**: `7 days` for hot backups; longer if your storage permits.
- **Storage**: Local volume by default. For real disaster recovery, configure an **S3 destination** (Coolify supports it natively) so a single VPS loss doesn't take backups down with the data.

<!-- screenshot 14 (nice-to-have): MongoDB resource → Backups tab with scheduled backup configured (Daily 02:00, retention 7 days). -->

For an ad-hoc manual backup. Coolify starts MongoDB with authentication enabled, so
`mongodump` needs credentials — without them it fails with `Authentication failed`.
Read them from the container's own environment so the password never lands in your
shell history:

```bash
docker exec <mongo-container> sh -c 'mongodump --archive --gzip \
  -u "$MONGO_INITDB_ROOT_USERNAME" \
  -p "$MONGO_INITDB_ROOT_PASSWORD" \
  --authenticationDatabase admin' > backup-$(date +%F).gz
```

To restore:

```bash
gunzip -c backup-2026-05-07.gz | docker exec -i <mongo-container> sh -c 'mongorestore --archive --drop \
  -u "$MONGO_INITDB_ROOT_USERNAME" \
  -p "$MONGO_INITDB_ROOT_PASSWORD" \
  --authenticationDatabase admin'
```

> **Note:** `--drop` wipes each collection before restoring it. Rehearse on a
> throwaway database name first, not on the live one.
>
> **Test the restore at least once** before you need it. A backup you've never restored is barely a backup.

## Troubleshooting

The five errors you're statistically likely to hit, in roughly the order people hit them.

### 1. `MongoNetworkError: connect ECONNREFUSED <mongo-host>:27017`

Two different problems produce this, and they need different fixes.

**The containers aren't on the same Docker network.** Coolify auto-resolves a database's
container name only for resources inside the *same project*. If the app and the MongoDB
resource live in different projects, the hostname never resolves and every connection is
refused, cold start or not. Simplest fix: move both into one project.

**MongoDB genuinely wasn't ready yet.** Coolify has no `depends_on` for Dockerfile-based
applications — that setting exists only for Docker Compose resources — so start ordering
isn't something you can configure away here. In practice the mongoose driver already
retries for `serverSelectionTimeoutMS` (30 s by default), which absorbs a normal Mongo
cold start. If your app still loses the race, deploy the MongoDB resource first, wait for
it to report healthy, then deploy the app.

### 2. `Error: connect ECONNREFUSED ::1:5432` (wrong DB type)

You forgot to set `DATABASE_URL`, or there's a typo in it. Payload's mongoose adapter falls back to a localhost lookup that resolves to nothing in the container. Triple-check the env var. The connection string from Coolify's MongoDB resource view is the canonical source.

### 3. Build OOMs ("Killed signal: 9") on a small VPS

Next.js production build briefly needs **2–3 GB RAM**. On a 4 GB VPS with MongoDB also running, the build can OOM. Two fixes:

- **Add swap** (cheap, works immediately):

  ```bash
  fallocate -l 2G /swapfile && chmod 600 /swapfile && mkswap /swapfile && swapon /swapfile
  echo '/swapfile none swap sw 0 0' >> /etc/fstab
  ```

- **Upgrade the VPS** to 8 GB RAM if your traffic justifies it.

### 4. Let's Encrypt rate-limited (5 duplicate certs / 7 days)

Triggered by repeated DNS or domain config errors during initial setup. Coolify uses LE production certs by default.

The limit you hit here is **"New Certificates per Exact Set of Identifiers"** — 5 certificates per 7 days for the *same* set of hostnames, which is exactly what retrying `cms.example.com` over and over does. The roomier limit of **50 certificates per registered domain per 7 days** is almost certainly not your problem.

If you hit it, you have two options:

- **Wait ~34 hours.** Capacity refills continuously — one duplicate certificate every 34 hours — so there is no weekly reset to sit out.
- Configure the **DNS-01 challenge with a wildcard cert** — covers all subdomains, only counts as one certificate. Worth doing anyway if you plan multiple environments (`staging.example.com`, `preview.example.com`, …).

### 5. Cloudflare 522 / 524 timeouts on long-running requests

Cloudflare's free plan times out at **100 seconds**. Long Payload imports, migrations, or heavy GraphQL queries can exceed that. Two workarounds:

- For one-off admin tasks, skip the proxy and work directly against the VPS.

  > **Careful:** the `output: 'standalone'` runtime image contains only `server.js`,
  > `.next/static`, `public` and the dependencies Next.js traced — there is no `pnpm`
  > and no `payload` CLI, so `docker exec <app-container> pnpm payload migrate` fails
  > with `pnpm: not found`.

  Run Payload migrations from a full local checkout instead, tunnelling to the
  database over SSH:

  ```bash
  # 1. find the Mongo container's address on the Docker network
  ssh root@<vps-ip> "docker inspect -f \
    '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <mongo-container>"

  # 2. forward it to your laptop and leave this running
  ssh -L 27017:<mongo-ip>:27017 root@<vps-ip>

  # 3. in a second terminal, run against the tunnel
  DATABASE_URL='mongodb://payload:<password>@127.0.0.1:27017/payload?authSource=admin' \
    pnpm payload migrate
  ```

  With the MongoDB adapter most schema changes need no migration at all — Payload
  writes documents in the new shape and reads tolerate the old one. Reach for
  migrations when you need a *data* transform, not a schema one.

- For app-level long requests, move the work to a Payload **job/queue** (`payload.jobs.queue(...)`) and return immediately — Cloudflare-friendly by design.

## Security checklist

Before sharing the URL publicly:

- [ ] **UFW or iptables**: allow only `22`, `80`, `443` inbound.
- [ ] **Disable root SSH login**, use key-based auth only:

  ```bash
  # /etc/ssh/sshd_config
  PermitRootLogin no
  PasswordAuthentication no
  PubkeyAuthentication yes
  ```

- [ ] **`fail2ban`** for SSH brute-force protection.
- [ ] **Coolify dashboard** behind a strong password + 2FA enabled.
- [ ] **`PAYLOAD_SECRET` ≥ 32 bytes**, never reused across environments.
- [ ] **MongoDB exposed only on the internal Docker network**, not on the host.
- [ ] **Cloudflare WAF rule** to block direct access to the origin IP (or set up an IP allow-list at the firewall level — only Cloudflare's [IP ranges](https://www.cloudflare.com/ips/) on `:443`).
- [ ] **Automated daily backups**, with at least one **test restore** documented.
- [ ] **Rotate secrets every 90 days** (`PAYLOAD_SECRET`, DB password, API tokens).
- [ ] **Restrict `cors`** in `payload.config.ts` to your real production origins — the starter uses `getServerSideURL()` which reads `NEXT_PUBLIC_SERVER_URL`, so this is correct as long as the env var is right.

## Performance tips

- **Enable swap** on a small VPS even if you're not currently OOMing — Linux makes intelligent use of it for caching.
- **Cloudflare cache rules** for static assets:
  - URL pattern `*/_next/static/*` → Cache TTL: 1 year, Edge TTL: 1 month.
  - URL pattern `*/api/*` → Bypass cache.
- **Next.js `output: 'standalone'`** (already set in the starter) — produces the smallest possible production image, ~150 MB vs ~600 MB for the default.
- **MongoDB indexes** are auto-created by Payload for collection-level queries, but verify with `db.<collection>.getIndexes()` in `mongosh` after adding new fields with frequent queries.
- **`docker system prune --volumes -f` monthly** to reclaim disk from old images and dangling layers — Coolify keeps the last 3 builds by default but old images add up.
- **Image optimization**: serve uploads through Next.js `<Image>` (already wired in the starter via `next/image` + Payload's `getURL` helper) — Cloudflare caches the optimized variants automatically.

## Going further

- **Object storage for media** — swap the local volume for S3-compatible storage. Payload's [`@payloadcms/plugin-cloud-storage`](https://payloadcms.com/docs/plugins/cloud-storage) wires this up in ~10 lines of config. Recommended providers: Hetzner Object Storage (€1/TB), Cloudflare R2 (free egress), Backblaze B2.
- **Transactional email** — plug Resend, Postmark or SendGrid into Payload's `email` config (any nodemailer-compatible adapter works). Without this, password reset / forgot-password flows are unavailable.
- **Multi-environment** — clone the application in Coolify pointing at a `develop` branch with `staging.example.com`. Use the same MongoDB resource with a different database name (`payload_staging`).
- **Monitoring** — Coolify integrates with Plausible / Umami for traffic; for app-level errors, plug Sentry into Next.js (`@sentry/nextjs`).
- **CI before deploy** — the [`nextjs-payload-starter`](https://github.com/DigitalVantage/nextjs-payload-starter) ships GitHub Actions that run lint + typecheck + tests + build on every PR; combined with branch protection, that means broken main never reaches Coolify.

## Useful links

- **Starter repo (the code half of this guide)**: <https://github.com/DigitalVantage/nextjs-payload-starter>
- **Coolify docs**: <https://coolify.io/docs>
- **Payload CMS docs**: <https://payloadcms.com/docs>
- **Next.js docs**: <https://nextjs.org/docs>
- **OVH VPS plans**: <https://www.ovhcloud.com/en/vps/>
- **Hetzner Cloud**: <https://www.hetzner.com/cloud>

## Contributing

Spotted an error, an outdated screenshot, a missing pitfall? **[Open an issue](https://github.com/DigitalVantage/coolify-nextjs-payload-guide/issues/new/choose)** or PR. See [CONTRIBUTING.md](CONTRIBUTING.md) for the workflow.

## License

[MIT](LICENSE) — Digital Vantage. Use, fork, translate, republish — just keep the attribution.

## Maintainer

**Digital Vantage** — partnerstwo technologiczne dla firm B2B w Polsce.

- Website: <https://www.digitalvantage.pl>
- Email: kontakt@digitalvantage.pl
- GitHub: [@DigitalVantage](https://github.com/DigitalVantage)

If you need a partner to deploy, scale or maintain a Payload + Next.js platform — [get in touch](https://www.digitalvantage.pl).
