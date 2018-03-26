# Project Workflows (WIP)
v2018.02.12 By @willfarrell

## New Repo
### Issue labeling (WIP)
- `git-labelmaker`
- label schemas

### Settings
#### Options
- [ ] Wikis
- [x] Restrict editing to users in teams with push access only
- [x] Issues
- [x] Allow forking
- [ ] Projects

#### Branches
Settings as per Developer Handbook

#### Integrations / Webhooks
TODO services from deve handbook

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
  - Security
    - [x] Require two-factor authentication for everyone in the Tardigrade Security organization.
  - Projects
    - [ ] Enable projects for the organization
    - [ ] Enable projects for all repositories
  - Teams
    - [ ] Enable team discussions for this organization
    
- Create GitHub infrastructure repo & setup
- Setup AWS Organization account & AWS env accounts w/ terraform

## New Project
- Create github app repo & setup, repeat if needed
- Setup ZenHub for repos
- Do Software Development Life Cycle analysis

