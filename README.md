# GitHub Release Demo

This repository automates GitHub Releases when `staging` is merged into `main`.

---

## What has been done

A GitHub Actions workflow at `[.github/workflows/release.yml](.github/workflows/release.yml)` runs after a **merged** pull request from `staging` into `main`. It:

1. Finds the latest `v*` tag (or starts from `v0.0.0`).
2. Bumps the patch version (`v1.0.3` → `v1.0.4`).
3. Lists commits between that tag and the merge commit.
4. Maps those commits to pull requests, excluding the `staging` → `main` PR itself.
5. Writes release notes (PRs with authors, a compare changelog link, and contributor profiles).
6. Tags `main` and publishes a GitHub Release.

---

## What problem it solves

When several PRs land on `staging` and ship together, it is easy to lose track of what went out, forget to tag, or paste a noisy commit log as the changelog.

This workflow:

- Records **what shipped** as PR numbers, titles, and developers
- Creates the **git tag and GitHub Release** on the merge that actually ships
- Keeps notes on the **GitHub Releases** page instead of a hand-written changelog

---

## How it works

```text
work merged into staging
        │
        │  open PR  staging → main  and merge it
        ▼
       main
        │
        ▼
  Release workflow
        │
        ├── next v* tag (patch + 1)
        ├── PRs since last tag
        ├── generated notes
        └── GitHub Release
```

1. Merge work into `staging`.
2. Open a pull request from `staging` to `main` and merge it.
3. The workflow runs only if the PR targeted `main`, was actually merged, and came from `staging`.
4. A new tag and GitHub Release appear on the repository.

---

## Limitations

- **Release trigger is `staging` → `main` only.** Merging `hotfix/`* (or any other branch) into `main`, or promoting `beta` / `sprint-*`, does not tag or publish a release.
- **Closed-but-unmerged PRs and direct pushes to `main` do nothing.**
- **Versioning is patch-only.** `v1.0.3` becomes `v1.0.4`. There is no major / minor bump from branch names or PR titles.
- `**package.json` is not updated.** Only the git tag and GitHub Release change.
- **Only `v`* tags are used.** A tag like `1.0.3` is ignored. The repo needs at least one real `v`* tag; a missing tag is treated as `v0.0.0`, which will fail `git log` if that tag does not exist.
- **Notes list PR number, title, and author.** Commits with no linked PR (direct commits, some cherry-picks) are omitted from the PR list. The `staging` → `main` PR itself is excluded. The Full Changelog compare link needs a real previous `v*` tag.
- **PR discovery uses `previous_tag..merge_commit`.** Rewritten history, force-pushes, or a shallow clone (`fetch-depth` other than `0`) can miss PRs.
- **Releases are published immediately.** There is no draft, approval, test, build, or deploy step. Merge one `staging` → `main` PR at a time and wait for the workflow to finish.

