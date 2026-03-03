# Release Strategies Demo

A docs-as-code demo comparing two GitHub release strategies.

## The Two Strategies

| | Strategy 1: Tag-Based Release | Strategy 2: Branch-Based Release |
|---|---|---|
| **Branch model** | Work directly on `main` | Work on `main`, merge to `release` |
| **Trigger** | Push a semver tag (`v1.0.0`) | PR merged into `release` branch |
| **Approval gate** | GitHub Environment with required reviewers | Pull Request review + branch protection |
| **Artifacts** | GitHub Release with built docs attached | GitHub Pages deployed on merge |
| **Docs** | [Strategy 1 Guide](docs/strategy-1-tag-based.md) | [Strategy 2 Guide](docs/strategy-2-branch-based.md) |

## Pros & Cons

### Strategy 1: Tag-Based Release

| Pros | Cons |
|---|---|
| Simple branch model — just `main`, no release branch to maintain | Approval happens _after_ code is on `main` — you're approving the release, not the change |
| Tags are immutable snapshots — easy to audit exactly what was released | Requires discipline around tagging — accidental tags trigger workflows |
| GitHub Environments provide a dedicated deployment review UI with history | Environment required reviewers not available on all plan tiers (private repos) |
| Artifacts attached directly to GitHub Releases — easy to download and distribute | Reviewers see a deployment approval screen, not a code diff |
| Auto-generated release notes from commits between tags | Harder to preview what will be included in a release before tagging |

### Strategy 2: Branch-Based Release (PR to Release)

| Pros | Cons |
|---|---|
| Reviewers see the full diff of what changed — approval happens _before_ it reaches release | Need to maintain a long-lived `release` branch |
| Uses the same PR workflow every developer already knows | Merge direction matters — must always go `main → release`, never reverse |
| Works on all GitHub plans — no special tier needed | If many changes accumulate on `main`, the PR diff can be overwhelming |
| Better audit trail — PR conversations, inline comments, and linked issues in one place | No dedicated deployment history UI like Environments provide |
| GitOps-friendly — branch state _is_ the source of truth for what's released | Version number is manual (in PR title) rather than derived from a tag |
| CI status checks validate the build before merge is allowed | Requires branch protection rules to enforce the approval gate |

## Quick Start

### Try Strategy 1 — Tag-Based Release

```bash
# Make changes on main
echo "new content" >> site/index.html
git add . && git commit -m "docs: update site content"
git push origin main

# Create a release tag
git tag v1.0.0
git push origin v1.0.0
# → Triggers approval workflow → creates GitHub Release with artifacts
```

### Try Strategy 2 — Branch-Based Release

```bash
# Make changes on main
echo "new content" >> site/index.html
git add . && git commit -m "docs: update site content"
git push origin main

# Open a PR from main → release
gh pr create --base release --head main \
  --title "Release v1.0.0" \
  --body "Docs updates for v1.0.0 release"
# → PR review for approval → merge deploys to Pages
```

## Repository Structure

```
.
├── README.md
├── docs/
│   ├── strategy-1-tag-based.md      # Deep dive: tag-based releases
│   └── strategy-2-branch-based.md   # Deep dive: branch-based releases
├── site/
│   └── index.html                   # Sample docs site (the "product")
└── .github/
    └── workflows/
        ├── tag-release.yml          # Strategy 1: build + release on tag
        └── branch-release.yml       # Strategy 2: build + deploy on merge to release
```

## Setup Requirements

To fully test both strategies, configure the following in your repo settings:

1. **Strategy 1** — Create a GitHub Environment called `production` with required reviewers
2. **Strategy 2** — Create a `release` branch and add branch protection rules requiring PR reviews
