# GitHub Release Demo

This repository automates GitHub Releases with **one version line per environment**:

| Branch | Tag line | Example |
| --- | --- | --- |
| `development` | alpha | `v1.0.0-alpha` |
| `staging` | beta | `v1.0.0-beta` |
| `main` | stable | `v1.0.0` |

Any merged PR into one of those branches bumps **only that branch’s tags**. Alpha, beta, and stable are independent.

---

## What has been done

A GitHub Actions workflow at [`.github/workflows/release.yml`](.github/workflows/release.yml) runs after **any merged pull request** into `development`, `staging`, or `main`:

| Target | New tag | Leaves unchanged |
| --- | --- | --- |
| `development` | next `vX.Y.Z-alpha` | beta and stable |
| `staging` | next `vX.Y.Z-beta` | alpha and stable |
| `main` | next `vX.Y.Z` | alpha and beta |

That includes hotfixes, sprint branches, and promotions (`development` → `staging`, `staging` → `main`).

For each merge it:

1. Finds the latest tag **for that channel only**.
2. Bumps the patch number and keeps the channel suffix (`-alpha`, `-beta`, or none).
3. Lists commits / PRs since that channel’s previous tag.
4. Writes release notes (PR number, title, developer; changelog compare; contributor profiles).
5. Tags the **target branch** and publishes a GitHub Release (alpha/beta as prereleases).

---

## What problem it solves

Each environment can move forward without rewriting another environment’s last release.

- A merge into `development` ships the next alpha without touching beta or production.
- A merge into `staging` (from `development`, a hotfix, or anything else) ships the next beta only.
- A merge into `main` (from `staging`, `hotfix/*`, or anything else) ships the next stable only.
- The changelog compare follows the **source branch** when that branch has its own tags.

---

## How it works

```text
any PR ──► development     tag: vX.Y.(Z+1)-alpha
any PR ──► staging         tag: vX.Y.(Z+1)-beta
any PR ──► main            tag: vX.Y.(Z+1)
```

1. Merge a PR into `development` → next **alpha** GitHub Release (prerelease).
2. Merge a PR into `staging` → next **beta** GitHub Release (prerelease).
3. Merge a PR into `main` → next **stable** GitHub Release.

Git tags cannot contain spaces, so tags are `v1.0.1-alpha` and `v1.0.1-beta` (not `v1.0.1 - alpha`).

### Full Changelog compare

The compare link is based on the **source branch** when that branch has a channel tag:

| Source → target | Compare |
| --- | --- |
| anything → `development` | previous alpha → new alpha |
| `development` → `staging` | latest alpha → new beta |
| `hotfix/*` (or other) → `staging` | previous beta → new beta |
| `staging` → `main` | latest beta → new stable |
| `hotfix/*` (or other) → `main` | previous stable → new stable |

If the source branch has no channel tag, the compare uses the previous tag of the **target** channel.

---

## Branch convention

| Branch | Role |
| --- | --- |
| `development` | Integration / alpha releases |
| `staging` | Pre-production / beta releases |
| `main` | Production / stable releases |
| `hotfix/*` | Urgent fixes; merging into `staging` or `main` still creates that branch’s next release |

---

## Limitations

- **Only merged PRs into `development`, `staging`, or `main` create a release.** Direct pushes do not. Closed-but-unmerged PRs do nothing.
- **Channels bump independently.** Alpha can be far ahead of beta/stable; promoting does not copy the alpha number onto beta.
- **Versioning is patch-only** within each channel.
- **`package.json` is not updated.**
- **Alpha/beta tags must end in `-alpha` / `-beta`.** Stable tags must be exactly `vX.Y.Z`. A missing channel tag is treated as `v0.0.0` plus that suffix; `git log` needs a real previous tag when one exists.
- **Notes list PR number, title, and author.** Commits with no linked PR may be omitted.
- **Releases are published immediately.** Alpha and beta are marked as GitHub prereleases. Merge one PR per target branch at a time so tags do not race.
