# Contributing

These workflows run on every pull request in [@FrancesCoronel](https://github.com/FrancesCoronel)'s repos, so changes here are reviewed carefully. Small, focused fixes are the easiest to merge. 🙏🏽

## What's welcome

- Bug fixes, such as a job that fails on a valid repo or skips a check it should run
- Security hardening, such as tighter permissions or safer handling of untrusted input
- Docs fixes in the README

For a new workflow or a new input, [open an issue](https://github.com/FrancesCoronel/workflows/issues/new/choose) first so we can agree on the shape.

## How to contribute

1. Fork the repo and create a branch off `main`
2. Make your change
3. Run `actionlint .github/workflows/*.yml templates/workflows/*.yml` (CI runs it too)
4. Open a pull request and fill in the template

## Ground rules

- Pin third-party actions to a full commit SHA, with the version in a comment. `actions/*` and `github/*` may use tags.
- Every workflow declares `permissions`, as narrow as the job allows.
- Never interpolate untrusted input (PR titles, branch names, issue text) straight into a `run:` script. Pass it through `env:` instead.
- If you add or rename an input, update the Inputs tables in the README and the files in `templates/`.
- Callers pin to `@v1`, so a change that breaks existing callers needs a new major tag.

## Security problems

Please don't open a public issue. Follow the [security policy](../SECURITY.md) instead.

## Code of Conduct

By taking part, you agree to follow the [code of conduct](CODE_OF_CONDUCT.md).
