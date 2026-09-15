# GitHub Release Demo

This repository automates GitHub Releases when a pull request is **merged into** `main`.

---

## What has been done

A GitHub Actions workflow at `[.github/workflows/release.yml](.github/workflows/release.yml)` runs after **any merged pull request into** `main`. It:

1. Finds the latest `v*` tag (or starts from `v0.0.0`).
2. Bumps the patch version (`v1.0.3` → `v1.0.4`).
3. Lists commits between that tag and the merge commit.
4. Maps those commits to pull requests, including the PR that was just merged.
5. Writes release notes with:
  - release PR number, title, and date
  - each included PR: number, title, and developer
  - a Full Changelog compare link (`previous tag` → `new tag`)
  - contributor profiles (avatars and GitHub links)
6. Tags `main` and publishes a GitHub Release.

---



## What problem it solves

When work lands on `main`, it is easy to lose track of what went out, forget to tag, or paste a noisy commit log as the changelog.

This workflow:

- Records **what shipped** as PR numbers, titles, and developers
- Creates the **git tag and GitHub Release** on every merge into `main`
- Adds a **compare link** so the full diff is one click away
- Lists **contributors** for the release
- Keeps notes on the **GitHub Releases** page instead of a hand-written changelog

---



## How it works

```text
any branch (staging, hotfix/*, sprint-*, …)
        │
        │  open PR → main  and merge it
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

1. Open a pull request targeting `main` from any branch.
2. Merge the pull request.
3. The workflow runs if the PR targeted `main` and was actually merged.
4. A new tag and GitHub Release appear on the repository.

## Limitations

- **Release trigger is a merged PR into** `main`**.** Direct pushes to `main` do not create a release. Closed-but-unmerged PRs do nothing.
- **Every merge into** `main` **publishes a new release.** Merge one PR at a time and wait for the workflow to finish so tags do not race.
- **Versioning is patch-only.** `v1.0.3` becomes `v1.0.4`. There is no major / minor bump from branch names or PR titles.
- `package.json` **is not updated.** Only the git tag and GitHub Release change.
- **Only** `v`* **tags are used.** A tag like `1.0.3` is ignored. The repo needs at least one real `v`* tag; a missing tag is treated as `v0.0.0`, which will fail `git log` if that tag does not exist.
- **Notes list PR number, title, and author.** Commits with no linked PR (direct commits, some cherry-picks) are omitted from the PR list. The Full Changelog compare link needs a real previous `v`* tag.
- **PR discovery uses** `previous_tag..merge_commit`**.** Rewritten history, force-pushes, or a shallow clone (`fetch-depth` other than `0`) can miss PRs.
- **Releases are published immediately.** There is no draft, approval, test, build, or deploy step.

