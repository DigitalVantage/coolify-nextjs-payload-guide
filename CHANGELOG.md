# Changelog

All notable changes to this guide will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
For a documentation repo "version" means: meaningful structural revisions, not every typo fix.

## [Unreleased]

## [0.1.0] - 2026-05-07

### Added

- Initial publication of the deployment guide.
- **10 step-by-step sections** covering: Coolify install, Cloudflare DNS, Coolify SSL, project + MongoDB, GitHub source, environment variables, persistent volume, deploy, Cloudflare proxy + Full (strict), auto-deploy webhook.
- **Operational sections**: database backups (scheduled + manual + restore), troubleshooting top 5, security checklist, performance tips, going-further roadmap.
- **Cross-link** to [`nextjs-payload-starter`](https://github.com/DigitalVantage/nextjs-payload-starter) — the working code that pairs with this guide.
- **CI**: `markdownlint-cli2` for style + `lychee` for link checking, run on every PR and push.
- **Dependabot** for GitHub Actions updates (weekly, Monday 06:00 Europe/Warsaw).
- **Community files**: `SECURITY.md`, `CODE_OF_CONDUCT.md` (Contributor Covenant 2.1), `CONTRIBUTING.md`, MIT `LICENSE`, GitHub PR template, structured Correction / Improvement issue forms with a contact-link routing card.

### Known limitations

- Screenshots of the Coolify UI not yet attached — placeholder spots in `docs/images/` to be filled from the maintainer's production deployment.
- Tested with Coolify 4.x, MongoDB 7, Payload 3.84, Next.js 16 — versions move fast; CI does not auto-detect when upstream UI changes break the guide.

[Unreleased]: https://github.com/DigitalVantage/coolify-nextjs-payload-guide/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/DigitalVantage/coolify-nextjs-payload-guide/releases/tag/v0.1.0
