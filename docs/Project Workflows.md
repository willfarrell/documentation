# Project Workflows (WIP)
v2018.02.12 By @willfarrell

## New Repo
### Issue labeling (WIP)
- `git-labelmaker`
- label schemas

### Document Templates
```
|-- .github
|   |-- ISSUE_TEMPLATE.md
|   |-- PULL_REQUEST_TEMPLATE.md
|-- docs
|   |-- images
|   |   |-- header.png
|   |-- CODE_OF_CONDUCT.md		# Create in GitHub to use template
|   |-- CONTRIBUTING.md
|   |-- GIT_COMMIT_TEMPLATE.md
|   |-- README.md
|-- CHANGELOG.md				# Build by code
|-- LICENSE

```

## New Client
- Create GitHub Organization
- Create GitHub infrastructure repo & setup
- Setup AWS Organization account & AWS env accounts w/ terraform

## New Project
- Create github app repo & setup, repeat if needed
- Setup ZenHub for repos

## New Feature Process
Stages refer to ZenHub triggers.

1. Create new branch from `develop` called `feature/{name}` [Stage: In Progress]
1. Develop feature
1. Push features to `origin/feature/{name}`
1. Create Pull Request (PR) to merge into [Stage: Code Review]
1. Assign a Reviewer on to the PR
1. Reviewer approves PR and merges [Stage: In QA]
1. New feature gets demoed to the team
1. Merge `develop` into `uat` or `v{major}` [Stage: In Client QA]
1. Client or Manager approves changes
1. Merge `uat` into `master` and tag with semver [Stage: Done]
