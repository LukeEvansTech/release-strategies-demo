# Strategy 1: Tag-Based Release

## Overview

In this strategy, all work happens on `main`. When you're ready to release, you push a semver git tag. The tag triggers a GitHub Actions workflow that builds artifacts, waits for manual approval via a GitHub Environment, and then publishes a GitHub Release with the built output attached.

```
main ──●──●──●──●──●──▶
                  │
                  └── tag: v1.0.0
                        │
                        ▼
              ┌──────────────────┐
              │  Actions: Build  │
              └────────┬─────────┘
                       ▼
              ┌──────────────────┐
              │  Approval Gate   │
              │  (Environment)   │
              └────────┬─────────┘
                       ▼
              ┌──────────────────┐
              │ GitHub Release   │
              │ + Artifacts      │
              └──────────────────┘
```

## How It Works

### 1. Develop on main

All documentation changes go directly to `main` via commits or merged PRs.

### 2. Tag a release

```bash
git tag v1.0.0
git push origin v1.0.0
```

### 3. Automated build

The workflow (`.github/workflows/tag-release.yml`) triggers on the tag push and:

- Checks out the code at the tagged commit
- Builds the docs site (stamps version into output)
- Uploads the built site as a workflow artifact

### 4. Approval gate

The `release` job uses a GitHub Environment called `production` configured with **required reviewers**. The workflow pauses and waits for an authorized reviewer to approve before continuing.

### 5. Publish

After approval, the workflow:

- Downloads the build artifact
- Packages it as a `.tar.gz`
- Creates a GitHub Release with auto-generated release notes
- Attaches the built docs artifact to the release

## Setup

### Create the GitHub Environment

1. Go to **Settings → Environments → New environment**
2. Name it `production`
3. Check **Required reviewers** and add the approvers
4. Optionally set a **wait timer** (e.g., 5 minutes) for a cooling-off period

### Branch protection (optional)

Add branch protection on `main` to require PRs before merging:

1. **Settings → Branches → Add rule** for `main`
2. Require pull request reviews before merging
3. Require status checks to pass

## Usage

```bash
# Work on docs
vim site/index.html
git add -A && git commit -m "docs: update API reference"
git push origin main

# Ready to release
git tag v1.2.0
git push origin v1.2.0

# Check the workflow
gh run list --workflow="Strategy 1: Tag-Based Release"

# After approval, view the release
gh release view v1.2.0
```

## Pros

- **Simple branch model** — single `main` branch, no long-lived release branch to maintain
- **Immutable snapshots** — tags point to exact commits, easy to audit
- **Rich approval UI** — GitHub Environments provide a dedicated review interface with deployment history
- **Artifact capture** — built output is attached to the GitHub Release, downloadable by anyone
- **Auto-generated notes** — GitHub generates changelogs from commits between tags

## Cons

- **Post-merge approval** — code is already on `main` when approval happens; you're approving the _release_, not the code change
- **Tag management** — requires discipline to tag correctly; accidental tags trigger workflows
- **Environment setup** — requires configuring GitHub Environments (not available on all plan tiers for private repos)
- **No diff view** — approvers see a deployment review, not a code diff
