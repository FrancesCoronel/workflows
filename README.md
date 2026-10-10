# workflows

Shared GitHub Actions for all of [@FrancesCoronel](https://github.com/FrancesCoronel)'s repos. Define once, call everywhere. ✨

Each repo keeps a tiny caller file. The real logic lives here, so a fix in this repo reaches every project at once.

## What's inside

| Workflow | What it does |
| --- | --- |
| [`ci-node.yml`](.github/workflows/ci-node.yml) | Detects npm, pnpm or yarn, installs from the lockfile, then runs whichever of `lint`, `typecheck` (or `tsc --noEmit`), `test` and `build` the repo has. Audits production deps for high and critical vulns (npm and pnpm). |
| [`ci-markdown.yml`](.github/workflows/ci-markdown.yml) | For docs and awesome-list repos. Runs markdownlint, then checks links with lychee. On PRs it only checks the files the PR touches, so a new link gets verified without old rot blocking it. |
| [`dependabot-automerge.yml`](.github/workflows/dependabot-automerge.yml) | Runs only after CI passes. Squash-merges Dependabot patch and minor bumps. Major bumps get a `major-update` label and a comment, then wait for a human. |
| [`coverage.yml`](.github/workflows/coverage.yml) | Runs `test:coverage` and reads the Istanbul `coverage-summary.json`. Fails below the `min` floor, and on PRs fails if lines, branches, functions or statements drop below the base branch. |
| [`web-quality.yml`](.github/workflows/web-quality.yml) | Builds and starts a web app, then runs pa11y-ci with axe (zero WCAG 2 AA violations) and Lighthouse CI (performance 90+, accessibility, best practices and SEO 100). |
| [`security.yml`](.github/workflows/security.yml) | Dependency review on PRs (no new vulnerable deps at any severity), zizmor on the workflows, and gitleaks for committed secrets. |
| [`ci-shell.yml`](.github/workflows/ci-shell.yml) | ShellCheck on every shell script, for dotfiles and install scripts. |
| [`codeql.yml`](.github/workflows/codeql.yml) | CodeQL security scanning, with languages as an input. |
| [`community-triage.yml`](.github/workflows/community-triage.yml) | Labels PRs from outside contributors as `community` so they get a careful review and never get auto-merged. |

## Add to a repo

1. Copy the files from [`templates/`](templates) into the repo's `.github/` folder:
   - `templates/workflows/pr.yml` → `.github/workflows/pr.yml` (Node CI + Dependabot auto-merge)
   - or `templates/workflows/pr-markdown.yml` → `.github/workflows/pr.yml` (Markdown CI + Dependabot auto-merge)
   - or `templates/workflows/pr-shell.yml` → `.github/workflows/pr.yml` (ShellCheck)
   - `templates/workflows/codeql.yml` → `.github/workflows/codeql.yml`
   - `templates/workflows/community.yml` → `.github/workflows/community.yml`
   - `templates/dependabot.yml` → `.github/dependabot.yml`
2. Delete any old workflows that these replace.
3. In the repo's **Settings → General**, turn on **Allow auto-merge** and **Automatically delete head branches**.
4. Optional, for public repos: add a branch ruleset on `main` that requires the `ci / ci` check. Auto-merge will then also wait for any other required checks, such as Vercel.

A caller looks like this:

```yaml
jobs:
  ci:
    uses: FrancesCoronel/workflows/.github/workflows/ci-node.yml@v1

  dependabot:
    needs: ci
    uses: FrancesCoronel/workflows/.github/workflows/dependabot-automerge.yml@v1
    permissions:
      contents: write
      pull-requests: write
```

## Inputs

**`ci-node.yml`**

| Input | Default | |
| --- | --- | --- |
| `node-version` | `""` | Empty uses `.nvmrc` or `.node-version` if present, else Node 24 |
| `pnpm-version` | `""` | Empty reads `packageManager` from `package.json` |
| `working-directory` | `.` | For monorepos or apps in a subfolder |
| `typecheck` | `true` | Set `false` while a repo has known type errors |
| `extra-scripts` | `""` | More scripts to run, e.g. `lint:md` |
| `run-build` | `true` | Set `false` if the build needs secrets that Dependabot PRs can't read |
| `audit` | `true` | `npm audit --omit=dev` or `pnpm audit --prod`, failing on high and critical |

**`ci-markdown.yml`**

| Input | Default | |
| --- | --- | --- |
| `lint` | `true` | markdownlint-cli2 |
| `lint-globs` | `**/*.md`, skipping `.github` and `node_modules` | One glob per line, `#` to exclude. Without a `.markdownlint*` config, line length, inline HTML and first-line-heading rules are off |
| `links` | `true` | lychee link check |
| `lychee-args` | `""` | e.g. `--exclude linkedin.com` |

**`dependabot-automerge.yml`**

| Input | Default | |
| --- | --- | --- |
| `merge-method` | `squash` | `squash`, `merge` or `rebase` |
| `allow-major` | `false` | Auto-merge majors too (only for repos with strong tests) |

**`coverage.yml`**

| Input | Default | |
| --- | --- | --- |
| `script` | `test:coverage` | Must write an Istanbul json-summary (vitest `--coverage.reporter=json-summary`, jest `--coverageReporters=json-summary`) |
| `summary-path` | `coverage/coverage-summary.json` | |
| `min` | `100` | Floor for every metric. A repo with few tests starts at `0` and raises it as tests land. The job summary says when the floor can go up |
| `target` | `100` | Shown in the job summary |
| `ratchet` | `true` | On PRs, also measure the base branch and fail on any drop |

With no `test:coverage` script the job counts coverage as 0% and warns, so `min: 0` passes and anything higher fails.

**`web-quality.yml`**

| Input | Default | |
| --- | --- | --- |
| `urls` | `/` | Paths to test, one per line |
| `build-script` / `start-script` | `build` / `start` | Empty `build-script` skips the build |
| `port` | `3000` | |
| `a11y` / `lighthouse` | `true` / `true` | |
| `a11y-standard` | `WCAG2AA` | |
| `pa11y-config` / `lighthouse-config` | `""` | Bring your own. A custom `lighthouserc.json` should leave out `startServerCommand`, since the server is already running |
| `performance-min` | `0.9` | Scores on shared runners move a few points between runs, so 1.0 would flake |
| `accessibility-min`, `best-practices-min`, `seo-min` | `1` | |

Secret `build-env`: optional dotenv lines written to `.env.local` before the build.

**`security.yml`**

| Input | Default | |
| --- | --- | --- |
| `dependency-review` | `true` | Needs the dependency graph. Private repos also need GitHub Advanced Security, so set `false` there |
| `fail-on-severity` | `low` | |
| `workflow-audit` | `true` | zizmor. Uses the repo's `zizmor.yml` if it has one; otherwise third-party actions must be SHA-pinned while `actions/*`, `github/*` and this repo's `@v1` may use tags |
| `zizmor-min-severity` | `medium` | |
| `secrets` | `true` | gitleaks. Scans full history on pushes and the PR merge commit on PRs |

**`ci-shell.yml`**

| Input | Default |
| --- | --- |
| `severity` | `style` |
| `exclude` | `""` (e.g. `SC1090,SC1091`) |

**`codeql.yml`**

| Input | Default |
| --- | --- |
| `languages` | `'["javascript-typescript","actions"]'` |
| `queries` | `""` (e.g. `security-and-quality`) |

## The quality bar

Every repo aims for the same bar. A gate that a repo can't meet yet starts where the repo is today and only moves up.

| | Gate | Bar |
| --- | --- | --- |
| Tests | `coverage.yml` | 100% lines, branches, functions and statements. Repos below that ratchet up and can never go down |
| Accessibility | `web-quality.yml` | Zero axe violations at WCAG 2 AA on every listed page, and a Lighthouse accessibility score of 100 |
| Performance | `web-quality.yml` | Lighthouse performance 90+, best practices and SEO 100 |
| Security | `security.yml`, `codeql.yml`, `ci-node.yml` audit | No new vulnerable deps, no high or critical vulns in production deps, no CodeQL alerts, no workflow findings, no committed secrets |

Coverage, accessibility and performance don't apply to Markdown and shell repos, so those run `security.yml` and their own linters.

## Good to know

- **Dependabot PRs can't read Actions secrets.** Any job that needs one (an AI review, Sentry upload) should skip bot PRs with `if: github.event.pull_request.user.type != 'Bot'`, or it will fail on every bump.
- **Merges made by the workflow don't trigger other workflows.** GitHub won't start new runs from a `GITHUB_TOKEN` push. Deploys through the Vercel or Netlify apps are unaffected.
- **Private repos can call these** because this repo is public.
- **Callers pin to `@v1`.** Changes land on `main` first. Once they look good, push `main` to the `release` branch (`git push origin main:release`) or run the [Move v1 tag](.github/workflows/tag.yml) workflow by hand, and every repo picks them up on its next run. Breaking changes get a new `v2` tag instead.
- **Third-party actions are pinned to commit SHAs** (with the version in a comment). Dependabot bumps them monthly in one grouped PR.
- **This repo lints itself.** [`lint.yml`](.github/workflows/lint.yml) runs actionlint on the workflows and templates, and Dependabot keeps the actions here up to date.
