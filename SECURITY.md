# Security Policy

## Supported Versions

Only the latest `v1` tag is maintained. Changes land on `main` first and reach callers once `v1` moves.

## Reporting a Vulnerability

Please **do not** open a public issue for security problems, such as a workflow that could leak secrets, run untrusted code or merge something it shouldn't.

Instead, use one of these:

- **GitHub Private Vulnerability Reporting** (preferred): use the [Report a vulnerability](https://github.com/FrancesCoronel/workflows/security/advisories/new) button on the Security tab.
- **Email**: [frances@francescoronel.com](mailto:frances@francescoronel.com)

Please include:

- A description of the vulnerability
- Steps to reproduce
- Potential impact
- Any suggested fix (optional)

I aim to respond within **72 hours** and will keep you updated while the issue is investigated and fixed.

## Scope

In scope: the reusable workflows in `.github/workflows/` and the caller templates in `templates/`.

Out of scope: the third-party actions these workflows call, which should be reported to their own maintainers.
