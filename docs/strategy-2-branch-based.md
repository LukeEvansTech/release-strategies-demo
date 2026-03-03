# Strategy 2: Branch-Based Release (PR to Release Branch)

## Overview

In this strategy, `main` is the working branch where all changes land. When you're ready to release, you open a Pull Request from `main` into a long-lived `release` branch. The PR provides a diff-based review and approval gate. On merge, a GitHub Actions workflow builds the docs and publishes a release.

```
main    ──●──●──●──●──●──▶
                     \
                      PR (review + approve)
                       \
release ────────────────●──▶
                        │
                        ▼
              ┌──────────────────┐
              │  Actions: Build  │
              │  + Deploy/Release│
              └──────────────────┘
```

## How It Works

### 1. Develop on main

All documentation changes go to `main` — either direct commits or feature-branch PRs merged into main.

### 2. Open a release PR

```bash
gh pr create --base release --head main \
  --title "Release v1.0.0" \
  --body "## Changes
- Updated API reference
- Added troubleshooting guide"
```

### 3. Review and approve

- The PR shows a full diff of everything that changed since the last release
- A validation workflow runs to confirm the docs build cleanly
- Designated reviewers approve the PR

### 4. Merge triggers release

When the PR is merged into `release`, the workflow:

- Builds the docs site with the version stamped in
- Packages and uploads the artifact
- Creates a GitHub Release with auto-generated notes
- Tags the release commit

## Setup

### Create the release branch

```bash
# From main, create the release branch
git checkout -b release
git push -u origin release
```

### Configure branch protection

1. Go to **Settings → Branches → Add rule** for `release`
2. Enable:
   - **Require a pull request before merging**
   - **Require approvals** (set to 1 or more)
   - **Require status checks to pass** — select the `Validate docs build` check
   - **Do not allow bypassing the above settings**
3. Optionally restrict who can push to the branch

### Create the validation workflow

The repo includes `.github/workflows/validate-pr.yml` which runs on PRs targeting `release`. It validates that the docs build successfully and uploads a preview artifact.

## Usage

```bash
# Work on docs
vim site/index.html
git add -A && git commit -m "docs: update API reference"
git push origin main

# Ready to release — open a PR
gh pr create --base release --head main \
  --title "Release v1.1.0" \
  --body "Docs updates for v1.1.0"

# Check PR status
gh pr checks

# After approval and merge, view the release
gh release view v1.1.0
```

## Pros

- **Code-level review** — reviewers see the full diff of what changed since the last release
- **Familiar workflow** — uses the same PR process developers already know
- **Pre-merge approval** — nothing reaches `release` without explicit approval
- **Built-in CI gate** — status checks validate the build before merge is allowed
- **Audit trail** — PR conversation, reviews, and comments are preserved
- **No extra infrastructure** — works with branch protection rules available on all GitHub plans

## Cons

- **Branch maintenance** — need to maintain a long-lived `release` branch
- **Merge direction matters** — must always merge `main → release`, never the reverse
- **Large diffs** — if many changes accumulate on `main`, the PR diff can be overwhelming
- **No dedicated deployment UI** — unlike Environments, there's no deployment history view
- **Manual version tracking** — version number lives in the PR title; requires discipline
