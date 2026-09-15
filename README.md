# GitHub Release Demo

This repository automates GitHub Releases when a pull request is **merged into** `main`.

Alpha (`development`) and beta (`staging`) releases are implemented in `[.github/workflows/release.yml](.github/workflows/release.yml)` but **commented out** until they are confirmed. They are not active.

---

## What has been done

A GitHub Actions workflow runs after **any merged pull request into** `main`. It:

1. Finds the latest stable `vX.Y.Z` tag (or starts from `v0.0.0`).
2. Bumps the patch version (`v1.0.3` → `v1.0.4`).
3. Lists commits between that tag and the merge commit.
4. Maps those commits to pull requests, including the PR that was just merged.
5. Writes release notes with:
  - release PR number, title, and date
  - each included PR: number, title, and developer
  - a Full Changelog compare link (previous stable → new stable)
  - contributor profiles (avatars and GitHub links)
6. Tags `main` and publishes a GitHub Release.

Per-channel tags (`vX.Y.Z-alpha` on `development`, `vX.Y.Z-beta` on `staging`) remain in the workflow as commented code, marked `Restore after confirmation`.

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
any branch
        │
        │  open PR → main  and merge it
        ▼
       main
        │
        ▼
  Release workflow
        │
        ├── next stable tag (patch + 1)
        ├── PRs since last stable tag
        ├── generated notes
        └── GitHub Release
```

1. Open a pull request targeting `main` from any branch (`staging`, `hotfix/*`, and so on).
2. Merge the pull request.
3. The workflow runs only if the PR targeted `main` and was actually merged.
4. A new stable tag and GitHub Release appear on the repository.

Merges into `development` or `staging` do **not** create a release while alpha/beta is commented out.

## Limitations

- **Only a merged PR into** `main` **creates a release.** Direct pushes to `main` do not. Closed-but-unmerged PRs do nothing.
- **Merges into** `development` **or** `staging` **do not release** until the commented alpha/beta blocks are restored.
- **Every merge into** `main` **publishes a new stable release.** Merge one PR at a time and wait for the workflow to finish so tags do not race.
- **Versioning is patch-only.** `v1.0.3` becomes `v1.0.4`. There is no major / minor bump.
- `package.json` **is not updated.** Only the git tag and GitHub Release change.
- **Only exact** `vX.Y.Z` **tags count as stable.** Tags like `v1.0.3-alpha` are ignored for the main bump. A missing stable tag is treated as `v0.0.0`; `git log` needs a real previous tag when one exists.
- **Notes list PR number, title, and author.** Commits with no linked PR may be omitted.
- **Releases are published immediately.** There is no draft, approval, test, build, or deploy step.

