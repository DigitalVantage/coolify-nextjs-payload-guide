# Security policy

This is a documentation repo, but documentation can have security impact too — bad advice in a deployment guide ends up in real production environments.

## Reporting a security issue with the guide

If something in this guide could lead a reader to a less-secure deployment (weak default secret, missing hardening step, dangerous shortcut), please **do not open a public issue** — email us first:

> **security@digitalvantage.pl**

Include:

- The section / step where the problem is.
- A description of the impact (what an attacker could do if a reader follows the current guidance).
- Suggested fix, if you have one.

We will acknowledge within **2 working days** (Europe/Warsaw) and ship a correction PR within a week, or sooner if the issue is critical.

## What is and isn't a vulnerability here

**Is** a security report:

- A copy-paste command in this guide that downgrades default security (e.g. exposing a port that shouldn't be exposed).
- A configuration we recommend that has a known CVE or hardening guideline against it.
- An out-of-date hardening checklist that misses recent best practice.

**Is not**:

- "Self-hosting is less secure than managed services." That's a trade-off, not a vulnerability.
- "You should also do X" without a concrete attack path. Open a regular issue / PR for those.
- Vulnerabilities **inside Coolify, Payload, Next.js, MongoDB, Cloudflare, or any other product** the guide references — report those upstream to the respective maintainers.

## Production hardening reminder

The guide ships with a [Security checklist](README.md#security-checklist) section. If you're deploying to production, treat the checklist as a minimum, not a maximum. Specifically:

- Generate strong `PAYLOAD_SECRET`, `CRON_SECRET`, `PREVIEW_SECRET` (`openssl rand -base64 32` or longer).
- Run MongoDB only on the internal Docker network — never expose `27017` to the public internet.
- Restrict `cors` in `payload.config.ts` to real production origins.
- Keep Coolify itself behind a strong password + 2FA.
- Test database restores at least once before relying on backups.
