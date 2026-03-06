# ADO → GitHub Enterprise Cloud (EMU) Migration Checklist

> **Companion to:** [ADO-to-GHEC-EMU-Migration-Plan.md](ADO-to-GHEC-EMU-Migration-Plan.md)  
> **Date:** March 6, 2026  
> **Instructions:** Check off each item as it is completed. Add assignee/date in the rightmost columns.

---

## Phase 0 — Pre-Migration Assessment & Planning

### Stakeholder Alignment

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 0.1 | Identify executive sponsor | ☐ | | |
| 0.2 | Identify migration lead (team/individual) | ☐ | | |
| 0.3 | Identify all affected teams and team leads | ☐ | | |
| 0.4 | Communicate migration rationale and timeline to all teams | ☐ | | |
| 0.5 | Establish migration communication channel (Teams/Slack) | ☐ | | |
| 0.6 | Define success criteria and acceptance criteria | ☐ | | |

### Inventory & Discovery

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 0.7 | Run `gh ado2gh inventory-report --ado-org YOUR_ADO_ORG` | ☐ | | |
| 0.8 | Document total number of ADO organizations | ☐ | | |
| 0.9 | Document total number of team projects per organization | ☐ | | |
| 0.10 | Document total number of repositories (active vs. archived) | ☐ | | |
| 0.11 | Document total number of pull requests across all repos | ☐ | | |
| 0.12 | Analyze repository sizes with `git-sizer` | ☐ | | |
| 0.13 | Identify repos using Git LFS | ☐ | | |
| 0.14 | Identify repos using Git submodules (especially cross-repo) | ☐ | | |
| 0.15 | Count Azure Pipelines — YAML vs. Classic | ☐ | | |
| 0.16 | Count Azure Boards work items | ☐ | | |
| 0.17 | Inventory Azure Artifacts feeds and package counts | ☐ | | |
| 0.18 | Assess wiki usage across projects | ☐ | | |
| 0.19 | Inventory service hooks and integrations | ☐ | | |
| 0.20 | Inventory third-party ADO extensions in use | ☐ | | |
| 0.21 | Inventory service connections (Azure, Docker, NuGet, etc.) | ☐ | | |
| 0.22 | Count variable groups and secrets | ☐ | | |
| 0.23 | Document branch policies per repository | ☐ | | |

### Repository Health Check

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 0.24 | Run `git-sizer --verbose` on all repositories | ☐ | | |
| 0.25 | Flag repos exceeding 10 GiB total blob size | ☐ | | |
| 0.26 | Flag repos with files > 100 MiB (plan Git LFS) | ☐ | | |
| 0.27 | Flag repos with non-ASCII characters in refs | ☐ | | |
| 0.28 | Identify all TFVC repositories | ☐ | | |

### TFVC Conversion (if applicable)

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 0.29 | Inventory all TFVC repos | ☐ | | |
| 0.30 | Convert TFVC repos to Git using `git-tfs` or ADO built-in import | ☐ | | |
| 0.31 | Validate TFVC → Git conversion completeness | ☐ | | |
| 0.32 | Plan adaptation for any TFVC-specific workflows | ☐ | | |

### Licensing

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 0.33 | Determine required GitHub Enterprise Cloud seat count | ☐ | | |
| 0.34 | Determine GitHub Actions minute requirements | ☐ | | |
| 0.35 | Determine GitHub Packages storage requirements | ☐ | | |
| 0.36 | Determine GitHub Advanced Security (GHAS) license needs | ☐ | | |
| 0.37 | Determine GitHub Copilot license needs | ☐ | | |
| 0.38 | Compare with current Azure DevOps licensing costs | ☐ | | |

### Batching Strategy

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 0.39 | Select batching strategy (Big Bang / by Org / by Team / by Criticality) | ☐ | | |
| 0.40 | Define batch composition (which repos in which batch) | ☐ | | |
| 0.41 | Define batch sequence and timeline | ☐ | | |
| 0.42 | Identify dependencies between batches | ☐ | | |
| 0.43 | Define rollback criteria for each batch | ☐ | | |

---

## Phase 1 — Enterprise & EMU Setup

### Enterprise Account

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 1.1 | Purchase GitHub Enterprise Cloud with EMU | ☐ | | |
| 1.2 | Receive enterprise short code from GitHub | ☐ | | |
| 1.3 | Complete account setup with GitHub account manager | ☐ | | |
| 1.4 | Decide hosting: github.com vs GHE.com (data residency) | ☐ | | |

### Enterprise Configuration

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 1.5 | Sign in as setup user (`SHORT-CODE_admin`) | ☐ | | |
| 1.6 | Set enterprise name and URL | ☐ | | |
| 1.7 | Set default repository visibility (private/internal) | ☐ | | |
| 1.8 | Configure repository creation permissions | ☐ | | |
| 1.9 | Configure forking policies | ☐ | | |
| 1.10 | Configure GitHub Actions policies | ☐ | | |
| 1.11 | Configure GitHub Packages policies | ☐ | | |
| 1.12 | Configure GitHub Copilot policies | ☐ | | |
| 1.13 | Configure 2FA requirements | ☐ | | |
| 1.14 | Configure SSH certificate authorities (if needed) | ☐ | | |
| 1.15 | Configure enterprise IP allow list (if needed) | ☐ | | |

### Authentication Protocol

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 1.16 | Select OIDC or SAML based on IdP | ☐ | | |
| 1.17 | Document the decision and rationale | ☐ | | |

---

## Phase 2 — Identity & Access Management

### IdP / SCIM Provisioning

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 2.1 | In Entra ID → Enterprise Applications → Add GitHub EMU app | ☐ | | |
| 2.2 | Configure SCIM provisioning mode to **Automatic** | ☐ | | |
| 2.3 | Set SCIM Tenant URL (`https://api.github.com/scim/v2/enterprises/ENTERPRISE`) | ☐ | | |
| 2.4 | Set Secret Token (PAT from setup user with `admin:enterprise`) | ☐ | | |
| 2.5 | Map IdP attributes (userName, emails, givenName, surname) | ☐ | | |
| 2.6 | Configure group provisioning for team mapping | ☐ | | |
| 2.7 | Test provisioning with a pilot group (5–10 users) | ☐ | | |
| 2.8 | Verify pilot users appear in GitHub enterprise | ☐ | | |
| 2.9 | Enable full provisioning | ☐ | | |

### SSO Configuration

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 2.10 | Configure SSO (OIDC or SAML) in Entra ID GitHub EMU app | ☐ | | |
| 2.11 | Set reply URL / ACS URL per GitHub docs | ☐ | | |
| 2.12 | Test SSO login with setup user | ☐ | | |
| 2.13 | Assign users and groups in Entra ID | ☐ | | |
| 2.14 | Verify all assigned users are provisioned on GitHub | ☐ | | |

### Group & Team Mapping

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 2.15 | Define Entra ID group → GitHub team mapping table | ☐ | | |
| 2.16 | Create Entra ID groups if they don't exist | ☐ | | |
| 2.17 | Assign users to appropriate Entra ID groups | ☐ | | |
| 2.18 | Configure IdP group → GitHub team synchronization | ☐ | | |
| 2.19 | Validate team membership after provisioning | ☐ | | |

### User Communication

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 2.20 | Communicate new EMU username format to all users | ☐ | | |
| 2.21 | Document how to find your GitHub managed username | ☐ | | |
| 2.22 | Update any automation that references old usernames | ☐ | | |

---

## Phase 3 — Tool Installation & Configuration

### CLI Setup

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 3.1 | Install GitHub CLI (`brew install gh` or equivalent) | ☐ | | |
| 3.2 | Verify GitHub CLI version ≥ 2.4.0 (`gh --version`) | ☐ | | |
| 3.3 | Install ADO2GH extension (`gh extension install github/gh-ado2gh`) | ☐ | | |
| 3.4 | Update ADO2GH to latest (`gh extension upgrade github/gh-ado2gh`) | ☐ | | |

### Credentials & Environment

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 3.5 | Create GitHub PAT (classic) with required scopes | ☐ | | |
| 3.6 | Create Azure DevOps PAT with required scopes | ☐ | | |
| 3.7 | Set `GH_PAT` environment variable | ☐ | | |
| 3.8 | Set `ADO_PAT` environment variable | ☐ | | |
| 3.9 | Set `TARGET_API_URL` env var (if using GHE.com) | ☐ | | |
| 3.10 | Allow ADO PAT to access all accessible organizations (if multi-org) | ☐ | | |

### Network & Access

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 3.11 | Add GitHub IP ranges to organization IP allow list | ☐ | | |
| 3.12 | Temporarily disable IdP CAP restrictions during migration | ☐ | | |
| 3.13 | Add "Repository migrations" to bypass list for all rulesets | ☐ | | |

---

## Phase 4 — Organizational Structure Design

### Structure Planning

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 4.1 | Map ADO organizations to GitHub organizations (1:1) | ☐ | | |
| 4.2 | Define GitHub organization naming convention | ☐ | | |
| 4.3 | Define repository naming convention | ☐ | | |
| 4.4 | Document full ADO → GitHub name mapping | ☐ | | |

### Organization Setup

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 4.5 | Create all GitHub organizations under the enterprise | ☐ | | |
| 4.6 | Configure organization settings (member privileges, default perms) | ☐ | | |
| 4.7 | Create teams within organizations based on IdP group mapping | ☐ | | |
| 4.8 | Link teams to IdP groups for automatic membership sync | ☐ | | |

### Repository Naming

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 4.9 | Decide on `--target-repo` renames (if standardizing names) | ☐ | | |
| 4.10 | Update migration script with `--target-repo` flags | ☐ | | |

---

## Phase 5 — Repository Migration (Trial Runs)

### Script Generation & Review

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 5.1 | Generate migration script with `gh ado2gh generate-script` | ☐ | | |
| 5.2 | Review generated `migrate.ps1` | ☐ | | |
| 5.3 | Remove/comment out repos not in scope | ☐ | | |
| 5.4 | Update `--target-repo` values for any renames | ☐ | | |
| 5.5 | Set `--target-repo-visibility` for each repo | ☐ | | |
| 5.6 | Verify batch size is manageable | ☐ | | |

### Trial Environment

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 5.7 | Create sandbox organizations (e.g., `acme-platform-sandbox`) | ☐ | | |
| 5.8 | Grant migrator role access to sandbox org | ☐ | | |

### Execute Trial

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 5.9 | Run trial migration against sandbox org | ☐ | | |

### Trial Validation

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 5.10 | All expected repos exist in target org | ☐ | | |
| 5.11 | Git history is complete (commit counts match) | ☐ | | |
| 5.12 | All branches migrated correctly | ☐ | | |
| 5.13 | All tags migrated correctly | ☐ | | |
| 5.14 | Pull requests migrated with all comments | ☐ | | |
| 5.15 | PR review comments preserved | ☐ | | |
| 5.16 | Work item links (AB#) preserved | ☐ | | |
| 5.17 | PR attachments accessible | ☐ | | |
| 5.18 | Branch policies migrated | ☐ | | |
| 5.19 | Large files handled correctly | ☐ | | |
| 5.20 | Migration logs reviewed — no critical warnings | ☐ | | |
| 5.21 | Teams created correctly | ☐ | | |
| 5.22 | Repository visibility is correct | ☐ | | |
| 5.23 | No ruleset violations | ☐ | | |
| 5.24 | Git clone/push/pull operations work | ☐ | | |
| 5.25 | Code search works (after re-indexing completes) | ☐ | | |

### Benchmarking & Sign-Off

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 5.26 | Record migration duration per repository | ☐ | | |
| 5.27 | Identify slow repositories (high PR count) | ☐ | | |
| 5.28 | Calculate total estimated production migration time | ☐ | | |
| 5.29 | Plan maintenance window based on estimates | ☐ | | |
| 5.30 | Document all issues found during trial | ☐ | | |
| 5.31 | Create remediation plan for each issue | ☐ | | |
| 5.32 | Re-run trial if significant changes were made | ☐ | | |
| 5.33 | Get stakeholder sign-off on trial results | ☐ | | |
| 5.34 | Delete sandbox organizations (cleanup) | ☐ | | |

---

## Phase 6 — Repository Migration (Production)

### Pre-Migration Go/No-Go

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 6.1 | All trial runs completed successfully | ☐ | | |
| 6.2 | Stakeholder sign-off obtained | ☐ | | |
| 6.3 | Go/No-Go communication sent to all affected teams | ☐ | | |
| 6.4 | Maintenance window scheduled and communicated | ☐ | | |
| 6.5 | ADO repos set to read-only (recommended) | ☐ | | |
| 6.6 | Rollback plan documented and tested | ☐ | | |
| 6.7 | On-call support team identified | ☐ | | |

### Execute Production Migration

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 6.8 | Run production migration script (`./migrate.ps1`) | ☐ | | |
| 6.9 | Monitor migration progress — check for failures | ☐ | | |
| 6.10 | Re-run failed migrations (if any) after fixing root cause | ☐ | | |
| 6.11 | Confirm all repos report `State: SUCCEEDED` | ☐ | | |

### Post-Migration Repository Tasks (per repo)

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 6.12 | Verify migration status (SUCCEEDED) for each repo | ☐ | | |
| 6.13 | Review migration log Issue in each repo | ☐ | | |
| 6.14 | Set repository visibility (private or internal) | ☐ | | |
| 6.15 | Configure branch protection rules / rulesets | ☐ | | |
| 6.16 | Set up CODEOWNERS file | ☐ | | |
| 6.17 | Configure required status checks | ☐ | | |
| 6.18 | Push Git LFS objects (clone → `lfs fetch ado --all` → `lfs push origin --all`) | ☐ | | |
| 6.19 | Update `.gitmodules` if using submodules (new GitHub URLs) | ☐ | | |

### Mannequin Reclaims

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 6.20 | Generate mannequin CSV (`gh ado2gh create-mannequin-csv`) | ☐ | | |
| 6.21 | Map mannequins to real GitHub managed users in CSV | ☐ | | |
| 6.22 | Run bulk mannequin reclaim (`gh ado2gh reclaim-mannequin --csv`) | ☐ | | |
| 6.23 | Verify user attribution is correct on migrated PRs | ☐ | | |
| 6.24 | Grant reclaimed users appropriate repository access (if not via teams) | ☐ | | |

---

## Phase 7 — CI/CD Pipeline Migration

### Strategy & Planning

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 7.1 | Decide pipeline strategy: Keep Azure Pipelines / Migrate to Actions / Hybrid | ☐ | | |
| 7.2 | Install GitHub Actions Importer (`gh extension install github/gh-actions-importer`) | ☐ | | |
| 7.3 | Run Actions Importer audit (`gh actions-importer audit azure-devops`) | ☐ | | |
| 7.4 | Review audit results — identify auto-convertible vs. manual pipelines | ☐ | | |
| 7.5 | Prioritize pipelines for migration (critical → low-risk) | ☐ | | |

### Pipeline Conversion

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 7.6 | Dry-run conversion for each pipeline | ☐ | | |
| 7.7 | Review converted workflows — fix manual conversion items | ☐ | | |
| 7.8 | Run `gh actions-importer migrate` for auto-convertible pipelines | ☐ | | |
| 7.9 | Manually rewrite Classic (non-YAML) pipelines as GitHub Actions | ☐ | | |
| 7.10 | Validate all converted workflows (syntax + execution) | ☐ | | |

### Self-Hosted Runners

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 7.11 | Inventory existing ADO self-hosted agents | ☐ | | |
| 7.12 | Plan runner infrastructure (VMs, Kubernetes/ARC, etc.) | ☐ | | |
| 7.13 | Install and register GitHub Actions self-hosted runners | ☐ | | |
| 7.14 | Apply runner labels matching workflow needs | ☐ | | |
| 7.15 | Configure runner groups for organization-level management | ☐ | | |
| 7.16 | Evaluate Actions Runner Controller (ARC) for autoscaling | ☐ | | |

### Secrets, Variables & Environments

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 7.17 | Inventory all ADO variable groups and secrets | ☐ | | |
| 7.18 | Map ADO secrets → GitHub Secrets (org / repo / environment) | ☐ | | |
| 7.19 | Create GitHub Secrets (`gh secret set`) | ☐ | | |
| 7.20 | Recreate service connections as OIDC federation or secret-based auth | ☐ | | |
| 7.21 | Set up GitHub Environments with deployment protection rules | ☐ | | |
| 7.22 | Validate all secrets are correctly referenced in new workflows | ☐ | | |

### Hybrid (if keeping Azure Pipelines temporarily)

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 7.23 | Configure Azure Pipelines GitHub service connection | ☐ | | |
| 7.24 | Update pipeline triggers to use GitHub webhook events | ☐ | | |
| 7.25 | Test pipeline execution against GitHub repos | ☐ | | |
| 7.26 | Define timeline for full Actions migration | ☐ | | |

### Pipeline Validation

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 7.27 | Trigger CI on a PR — verify build passes | ☐ | | |
| 7.28 | Trigger CD / deployment — verify successful deploy | ☐ | | |
| 7.29 | Verify artifact publishing works | ☐ | | |
| 7.30 | Verify environment approvals / gates work | ☐ | | |
| 7.31 | Verify notifications (Slack, Teams, email) fire correctly | ☐ | | |

---

## Phase 8 — Work Item & Project Management

### Strategy Decision

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 8.1 | Decide approach: Keep Azure Boards / Migrate to GitHub Issues+Projects / Other | ☐ | | |

### Option A: Keep Azure Boards + GitHub Integration

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 8.2 | Install Azure Boards app for GitHub | ☐ | | |
| 8.3 | Connect Azure Boards orgs to GitHub orgs | ☐ | | |
| 8.4 | Configure `AB#` link syntax in commits and PRs | ☐ | | |
| 8.5 | Verify bidirectional work item links | ☐ | | |
| 8.6 | Train developers on `AB#1234` linking syntax | ☐ | | |

### Option B: Migrate to GitHub Issues + Projects

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 8.7 | Map ADO work item types → GitHub Issue types | ☐ | | |
| 8.8 | Map area paths → GitHub Labels | ☐ | | |
| 8.9 | Map iteration paths → GitHub Milestones | ☐ | | |
| 8.10 | Map custom fields → GitHub Projects custom fields | ☐ | | |
| 8.11 | Build or select migration script/tool | ☐ | | |
| 8.12 | Run work item migration | ☐ | | |
| 8.13 | Validate migrated issues (count, content, links) | ☐ | | |
| 8.14 | Set up GitHub Projects boards/views | ☐ | | |
| 8.15 | Redirect old work item references to new GitHub Issues | ☐ | | |

---

## Phase 9 — Packages & Artifacts Migration

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 9.1 | Inventory all Azure Artifacts feeds (NuGet, npm, Maven, PyPI, Universal) | ☐ | | |
| 9.2 | Identify consumers per feed (pipelines, dev machines) | ☐ | | |
| 9.3 | Set up GitHub Packages registries for each package type | ☐ | | |
| 9.4 | Re-publish packages to GitHub Packages | ☐ | | |
| 9.5 | Update `nuget.config` — new registry URL and credentials | ☐ | | |
| 9.6 | Update `.npmrc` — new registry scope and auth | ☐ | | |
| 9.7 | Update `settings.xml` (Maven) — new repository URL | ☐ | | |
| 9.8 | Update `pip.conf` / `pyproject.toml` — new index URL (if applicable) | ☐ | | |
| 9.9 | Update CI/CD pipelines to publish to GitHub Packages | ☐ | | |
| 9.10 | Set up upstream proxying (optional) | ☐ | | |
| 9.11 | Test package restore/install from new registries | ☐ | | |
| 9.12 | Deprecate Azure Artifacts feeds | ☐ | | |

---

## Phase 10 — Security & Compliance (GHAS)

### GitHub Advanced Security Enablement

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 10.1 | Enable GHAS at enterprise level | ☐ | | |
| 10.2 | Configure default CodeQL code scanning setup | ☐ | | |
| 10.3 | Enable secret scanning for all organizations | ☐ | | |
| 10.4 | Enable push protection for secret scanning | ☐ | | |
| 10.5 | Configure Dependabot security updates | ☐ | | |
| 10.6 | Add `dependabot.yml` to repositories | ☐ | | |
| 10.7 | Set up `SECURITY.md` security policy in each org | ☐ | | |
| 10.8 | Configure security advisories workflow | ☐ | | |
| 10.9 | Review Security Overview dashboard | ☐ | | |

### Governance & Compliance

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 10.10 | Configure repository rulesets at org/enterprise level | ☐ | | |
| 10.11 | Configure signed commit requirements (if applicable) | ☐ | | |
| 10.12 | Set up audit log streaming to SIEM | ☐ | | |
| 10.13 | Set up custom repository properties for compliance tracking | ☐ | | |
| 10.14 | Configure required workflows (if applicable) | ☐ | | |
| 10.15 | Review and set GitHub Actions allowed-actions list | ☐ | | |

---

## Phase 11 — Post-Migration Validation

### Infrastructure Verification

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 11.1 | Verify migration status for ALL repos | ☐ | | |
| 11.2 | Review ALL migration logs for errors/warnings | ☐ | | |
| 11.3 | Verify all repos have correct visibility | ☐ | | |
| 11.4 | Verify all mannequins are reclaimed | ☐ | | |
| 11.5 | Verify team access and permissions are correct | ☐ | | |
| 11.6 | Verify branch protection rules are applied | ☐ | | |

### Network & Access Cleanup

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 11.7 | Remove GEI IP ranges from allow lists | ☐ | | |
| 11.8 | Re-enable IdP CAP restrictions | ☐ | | |
| 11.9 | Remove "Repository migrations" from ruleset bypass lists | ☐ | | |

### Developer Workstation Updates

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 11.10 | Distribute instructions for `git remote set-url origin` | ☐ | | |
| 11.11 | Distribute HTTPS auth instructions (Git credential manager) | ☐ | | |
| 11.12 | Distribute SSH key setup instructions for GitHub | ☐ | | |
| 11.13 | Update IDE integrations (VS Code, Visual Studio, JetBrains) | ☐ | | |
| 11.14 | Update local `.npmrc`, `nuget.config`, etc. for new registries | ☐ | | |

### Integration Updates

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 11.15 | Update CI/CD service connections to point to GitHub | ☐ | | |
| 11.16 | Update webhook consumers (Slack, Teams, PagerDuty, etc.) | ☐ | | |
| 11.17 | Update ChatOps integrations | ☐ | | |
| 11.18 | Update monitoring and alerting systems | ☐ | | |
| 11.19 | Update internal documentation and wiki references | ☐ | | |
| 11.20 | Update scripts referencing ADO repository URLs | ☐ | | |

### Smoke Testing (per repo)

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 11.21 | Clone the repository from GitHub | ☐ | | |
| 11.22 | Create a feature branch | ☐ | | |
| 11.23 | Make a change, commit, and push | ☐ | | |
| 11.24 | Create a pull request | ☐ | | |
| 11.25 | Verify CI runs on the PR | ☐ | | |
| 11.26 | Merge the PR | ☐ | | |
| 11.27 | Verify deployment pipeline triggers (if applicable) | ☐ | | |

---

## Phase 12 — Developer Onboarding & Training

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 12.1 | Create internal migration FAQ document | ☐ | | |
| 12.2 | Create "Getting Started with GitHub (EMU)" guide | ☐ | | |
| 12.3 | Create ADO → GitHub cheat sheet (concept mapping) | ☐ | | |
| 12.4 | Document Git remote URL update instructions | ☐ | | |
| 12.5 | Create CODEOWNERS templates for teams | ☐ | | |
| 12.6 | Document GitHub Actions best practices for the org | ☐ | | |
| 12.7 | Set up `#github-migration-support` communication channel | ☐ | | |
| 12.8 | Schedule office hours for migration questions | ☐ | | |
| 12.9 | Run training: GitHub Fundamentals (UI, PR flow, Issues) | ☐ | | |
| 12.10 | Run training: EMU Authentication & Account | ☐ | | |
| 12.11 | Run training: GitHub Actions vs Azure Pipelines | ☐ | | |
| 12.12 | Run training: GitHub Advanced Security | ☐ | | |
| 12.13 | Run training: GitHub Copilot | ☐ | | |
| 12.14 | Run training: GitHub Projects (for PMs) | ☐ | | |
| 12.15 | Run training: GitHub API & Automation (for platform team) | ☐ | | |
| 12.16 | Run training: CODEOWNERS & Branch Protection (for team leads) | ☐ | | |
| 12.17 | Run training: GitHub Packages (for package maintainers) | ☐ | | |

---

## Phase 13 — Decommission Azure DevOps

### Coexistence Period (30–90 days)

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 13.1 | Keep ADO repos in read-only mode | ☐ | | |
| 13.2 | Monitor for lingering references to ADO repos | ☐ | | |
| 13.3 | Confirm all integrations point to GitHub | ☐ | | |
| 13.4 | Verify no active builds running against ADO repos | ☐ | | |
| 13.5 | Keep Azure Boards accessible (if still in use) | ☐ | | |

### Final Decommission

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 13.6 | Final confirmation: all repos migrated and validated | ☐ | | |
| 13.7 | Final confirmation: all pipelines running on GitHub (or Azure Pipelines + GitHub) | ☐ | | |
| 13.8 | Final confirmation: all packages available from new registries | ☐ | | |
| 13.9 | Export remaining data (wikis, test plans, dashboards) | ☐ | | |
| 13.10 | Archive ADO organizations (set to read-only) | ☐ | | |
| 13.11 | Revoke ADO PATs created for migration | ☐ | | |
| 13.12 | Remove ADO service connections no longer needed | ☐ | | |
| 13.13 | Update DNS/redirects (if applicable) | ☐ | | |
| 13.14 | Cancel/downgrade Azure DevOps licenses | ☐ | | |

### Data Retention

| # | Task | Done | Assignee | Date |
|---|---|---|---|---|
| 13.15 | Determine data retention requirements for ADO data | ☐ | | |
| 13.16 | Export audit logs from Azure DevOps | ☐ | | |
| 13.17 | Archive ADO organization (do NOT delete yet) | ☐ | | |
| 13.18 | Set calendar reminder for final deletion (e.g., 1 year) | ☐ | | |
| 13.19 | Document the final state and archive migration records | ☐ | | |

---

## Summary

| Phase | Items | Completed |
|---|---|---|
| Phase 0 — Assessment & Planning | 43 | ☐ /43 |
| Phase 1 — Enterprise & EMU Setup | 17 | ☐ /17 |
| Phase 2 — Identity & Access | 22 | ☐ /22 |
| Phase 3 — Tool Installation | 13 | ☐ /13 |
| Phase 4 — Org Structure | 10 | ☐ /10 |
| Phase 5 — Trial Migrations | 34 | ☐ /34 |
| Phase 6 — Production Migration | 24 | ☐ /24 |
| Phase 7 — CI/CD Migration | 31 | ☐ /31 |
| Phase 8 — Work Items | 15 | ☐ /15 |
| Phase 9 — Packages | 12 | ☐ /12 |
| Phase 10 — Security (GHAS) | 15 | ☐ /15 |
| Phase 11 — Post-Migration Validation | 27 | ☐ /27 |
| Phase 12 — Onboarding & Training | 17 | ☐ /17 |
| Phase 13 — Decommission ADO | 19 | ☐ /19 |
| **TOTAL** | **299** | |
