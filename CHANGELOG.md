# Changelog

All notable changes to this guide will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
For a documentation repo "version" means: meaningful structural revisions, not every typo fix.

## [Unreleased]

## [0.1.0] - 2026-08-25

First public release.

### Added

- **10 step-by-step sections** covering: Coolify install, Cloudflare DNS, Coolify SSL, project + MongoDB, GitHub source, environment variables, persistent volume, deploy, Cloudflare proxy + Full (strict), auto-deploy webhook.
- **Operational sections**: database backups (scheduled + manual + restore), troubleshooting top 5, security checklist, performance tips, going-further roadmap.
- **Article versions** of the guide, in [Polish](https://www.digitalvantage.pl/artykuly/strony-internetowe/technologie/self-hosting-coolify) and [English](https://www.digitalvantage.eu/posts/websites-internet/technologies/self-hosting-coolify).
- **Cross-link** to [`nextjs-payload-starter`](https://github.com/DigitalVantage/nextjs-payload-starter) — the working code that pairs with this guide.
- **CI**: `markdownlint-cli2` for style + `lychee` for link checking, run on every PR and push under a least-privilege workflow token.
- **Dependabot** for GitHub Actions updates (weekly, Monday 06:00 Europe/Warsaw).
- **Community files**: `SECURITY.md`, `CODE_OF_CONDUCT.md` (Contributor Covenant 2.1), `CONTRIBUTING.md`, MIT `LICENSE`, GitHub PR template, structured Correction / Improvement issue forms with a contact-link routing card.

### Known limitations

- Screenshots of the Coolify UI not yet attached — placeholder spots in `docs/images/` to be filled from the maintainer's production deployment.
- Written from a production deployment on Coolify 4.x, Payload 3.8x and Next.js 16. The MongoDB steps target **8.0**, the current supported line; the original deployment ran 7 and the Coolify flow is identical between the two.
- Versions move fast and CI does not auto-detect when an upstream UI change breaks the guide — corrections via issues are welcome.

[Unreleased]: https://github.com/DigitalVantage/coolify-nextjs-payload-guide/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/DigitalVantage/coolify-nextjs-payload-guide/releases/tag/v0.1.0
