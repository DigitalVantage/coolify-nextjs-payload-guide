# Screenshot capture cheatsheet

This is the working list of screenshots needed for the main `README.md`. Take them once during a real Coolify deployment, anonymize, save here as WebP, and replace the corresponding `<!-- screenshot NN -->` markers in the main README with the actual `![alt](docs/images/NN-...)` line.

## Conventions

- **Format**: WebP, max **200 KB** per image.
- **Naming**: `NN-short-slug.webp` where `NN` matches the screenshot number below (zero-padded so they sort correctly).
- **Resolution**: viewport ~1440×900, then crop to the relevant area (~800–1200 × 400–600 px output).
- **Theme**: pick **dark** or **light** for Coolify and stay consistent across all images. Recommended: **dark** (renders better on GitHub).
- **Browser**: clean profile / incognito, no extension noise.

## Privacy checklist (do this BEFORE committing each image)

Every screenshot must have:

- [ ] Real VPS IP address replaced with `198.51.100.10` (RFC 5737 documentation IP) or masked as `XXX.XXX.XXX.XXX`.
- [ ] Real production domain replaced with `cms.example.com` / `coolify.example.com`. Test/demo domains (e.g. `coolify-demo.digitalvantage.pl`) are fine to show.
- [ ] MongoDB password masked. Coolify normally redacts to `••••••`, but the connection-string view leaks it in plaintext on copy — mask it manually.
- [ ] `PAYLOAD_SECRET`, `CRON_SECRET`, `PREVIEW_SECRET` values masked.
- [ ] Any GitHub OAuth token / API key masked.
- [ ] Container name hash (e.g. `mongo-abc123def`) — optional to mask, low sensitivity but a small info-leak.
- [ ] Email address in admin signup view masked (`name@••••••`).
- [ ] Personal GitHub avatar / username in OAuth screens. The `DigitalVantage` org is OK to show; a maintainer's personal handle should be cropped out.

## Tools

- **macOS**: `Cmd+Shift+5` → annotate in Preview → export as WebP via `cwebp -q 85` (`brew install webp`). Preview's "Markup → Redact" makes the black bar.
- **Linux**: Flameshot (`apt install flameshot`) has built-in redact + arrows; pipe to `cwebp -q 85`.
- **Windows**: Snipping Tool / Snip & Sketch → online or `cwebp` for WebP conversion.

## Must-have screenshots (10 — block release without them)

| #  | README anchor                 | What to capture                                                                                                                                                                                                                                                                                          |
| -- | ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 01 | Step 1 (post-install)         | Coolify **Register / first admin signup** screen — the very first page you see at `http://<vps-ip>:8000`. No data filled in.                                                                                                                                                                             |
| 02 | Step 2                        | **Cloudflare → DNS → Records** page showing both `coolify` and `cms` A records with **gray cloud** (DNS-only). VPS IP masked.                                                                                                                                                                            |
| 03 | Step 3                        | **Coolify → Settings → Configuration** with the `Instance Domain` field set to `https://coolify.example.com` (or your demo domain).                                                                                                                                                                      |
| 04 | Step 4 (form)                 | **Coolify → + New Resource → Database → MongoDB** form, fields filled: username `payload`, database name `payload`, password masked.                                                                                                                                                                     |
| 05 | Step 4 (post-deploy)          | The MongoDB resource view showing the **Connection String** field. Mask password and container hash if you want.                                                                                                                                                                                          |
| 06 | Step 5 (GitHub App)           | The **Coolify GitHub App install permission** screen. The `DigitalVantage` org row visible.                                                                                                                                                                                                              |
| 07 | Step 5 (application)          | The **application creation form** with: Build Pack: Dockerfile, Domain: `https://cms.example.com`, Port Exposes: 3000, Branch: main.                                                                                                                                                                     |
| 08 | Step 6                        | The **Environment Variables** tab with all six vars added. The **Build Variable / Runtime Variable** toggle visible and the **Secret lock icon** on at least one of `PAYLOAD_SECRET` / `CRON_SECRET` / `PREVIEW_SECRET`. Values masked.                                                                  |
| 09 | Step 7                        | The **Persistent Storage / Storages** form filled in: Mount Path `/app/public/media`, Type Volume, Volume Name `payload-media`.                                                                                                                                                                          |
| 10 | Step 8                        | **Deploy logs** in success state — green status bar, last lines showing `Container started`, `Health check passed`, and the route added by Traefik.                                                                                                                                                       |

## Nice-to-have screenshots (4 — capture if you're already in that screen)

| #  | README anchor                 | What to capture                                                                                                                                                                                                                                                                                          |
| -- | ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 11 | Step 9                        | **Cloudflare → SSL/TLS → Overview** with encryption mode set to `Full (strict)` (highlighted).                                                                                                                                                                                                            |
| 12 | Step 9                        | The same Cloudflare DNS table as `02` but **after the flip**: `cms.example.com` now has the **orange cloud** (Proxied). A side-by-side or stacked comparison with `02` would be ideal but not required.                                                                                                  |
| 13 | Step 10                       | The application's **Webhooks** tab in Coolify (or GitHub repo Settings → Webhooks → Recent Deliveries) showing the auto-deploy webhook firing on push.                                                                                                                                                    |
| 14 | Database backups              | The MongoDB resource → **Backups** tab with a scheduled backup configured (`Daily 02:00`, retention `7 days`).                                                                                                                                                                                            |

## Workflow when you have the captures

1. Drop the WebP files in this directory with the exact filenames listed above.
2. In `README.md`, replace each `<!-- screenshot NN: <description> -->` HTML comment with:

   ```markdown
   ![<short alt text>](docs/images/NN-short-slug.webp)
   ```

3. Add a `[Unreleased]` entry to `CHANGELOG.md` under `### Added`: `- 14 screenshots illustrating the Coolify UI for steps 1–10 (must-have) and steps 9–10 + backups (nice-to-have).`
4. Open a PR titled `docs: add Coolify UI screenshots`. CI runs `markdownlint` and `lychee`; both pass on image-only changes.
5. Once merged, flip the repo back to public:

   ```bash
   gh repo edit DigitalVantage/coolify-nextjs-payload-guide --visibility public
   ```

6. Open a follow-up PR in [`nextjs-payload-starter`](https://github.com/DigitalVantage/nextjs-payload-starter) re-adding the cross-link callout that was reverted in #12.
