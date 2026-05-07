# Contributing

Thanks for your interest in `coolify-nextjs-payload-guide`. This is a documentation repo — most contributions are typo fixes, technical corrections, screenshot additions, or new troubleshooting entries from your own deployment experience.

## Code of conduct

Be excellent to each other. Personal attacks, harassment or discrimination of any kind are unacceptable. Maintainers may close issues and PRs at their discretion. See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## What kind of contributions we want

**Yes, please:**

- Typo fixes, grammar improvements, broken links.
- Technical corrections backed by your own deployment experience.
- Additional troubleshooting entries (with the actual error message you saw).
- Screenshots of the Coolify UI for steps that currently lack them.
- Translations (open an issue first to coordinate).
- Updates when Coolify, Payload or Next.js change in ways that break the guide.

**Probably not:**

- "Why don't you cover Vercel deployment too?" — out of scope. This guide is about Coolify.
- Speculative additions you haven't tried yourself. If you wouldn't run it in production, don't put it in the guide.
- Style-only rewrites of correct content.

## Reporting an issue

Open an issue using the **Correction** or **Improvement** template. Include:

- The section / step number.
- What's wrong or missing.
- The Coolify version, Payload version, Next.js version you observed it on.

If you spotted a security problem in the guide (e.g. an example secret check that's actually weak), email **security@digitalvantage.pl** instead of opening a public issue. See [SECURITY.md](SECURITY.md).

## Submitting a pull request

For typo / one-line fixes — just open the PR.

For larger changes (new section, restructure, translations) — **open an issue first** so we can align on scope before you spend an hour writing.

### PR checklist

- [ ] CI is green (`markdownlint` + `lychee` link checker).
- [ ] You actually tested any commands or config snippets you added.
- [ ] CHANGELOG.md updated under `[Unreleased]` for user-visible changes.
- [ ] Screenshots (if any) compressed (<200 KB each, PNG or WebP).

### Commit conventions

[Conventional Commits](https://www.conventionalcommits.org/):

```text
<type>(<scope>): short imperative description

Optional body explaining WHY.
```

**Types**: `fix`, `feat`, `docs`, `chore`, `style`, `refactor`, `ci`. For this repo most commits will be `docs:` or `fix:`.

### Branching

- `main` — always shippable, branch-protected.
- Feature/fix branches: `docs/<short-description>` or `fix/<short-description>`.

## License of contributions

By submitting a PR you agree that your contribution is licensed under the [MIT license](LICENSE) of this project.
