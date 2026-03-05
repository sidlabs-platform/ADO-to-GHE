# Azure DevOps to GitHub Enterprise Cloud (EMU) — Comprehensive Migration Plan

> **Version:** 1.0  
> **Date:** March 6, 2026  
> **Scope:** Full migration of Azure DevOps Cloud repositories, pipelines, boards, packages, and identity to GitHub Enterprise Cloud with Enterprise Managed Users (EMU)

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Migration Scope & What Moves](#2-migration-scope--what-moves)
3. [Phase 0 — Pre-Migration Assessment & Planning](#3-phase-0--pre-migration-assessment--planning)
4. [Phase 1 — Enterprise & EMU Setup](#4-phase-1--enterprise--emu-setup)
5. [Phase 2 — Identity & Access Management](#5-phase-2--identity--access-management)
6. [Phase 3 — Tool Installation & Configuration](#6-phase-3--tool-installation--configuration)
7. [Phase 4 — Organizational Structure Design](#7-phase-4--organizational-structure-design)
8. [Phase 5 — Repository Migration (Trial Runs)](#8-phase-5--repository-migration-trial-runs)
9. [Phase 6 — Repository Migration (Production)](#9-phase-6--repository-migration-production)
10. [Phase 7 — CI/CD Pipeline Migration (Azure Pipelines → GitHub Actions)](#10-phase-7--cicd-pipeline-migration)
11. [Phase 8 — Work Item & Project Management Migration (Azure Boards → GitHub Issues/Projects)](#11-phase-8--work-item--project-management-migration)
12. [Phase 9 — Packages & Artifacts Migration](#12-phase-9--packages--artifacts-migration)
13. [Phase 10 — Security & Compliance (GHAS)](#13-phase-10--security--compliance-ghas)
14. [Phase 11 — Post-Migration Tasks & Validation](#14-phase-11--post-migration-tasks--validation)
15. [Phase 12 — Developer Onboarding & Training](#15-phase-12--developer-onboarding--training)
16. [Phase 13 — Decommission Azure DevOps](#16-phase-13--decommission-azure-devops)
17. [Risk Register & Mitigations](#17-risk-register--mitigations)
18. [RACI Matrix](#18-raci-matrix)
19. [Key Differences: Azure DevOps vs GitHub](#19-key-differences-azure-devops-vs-github)
20. [Reference Commands & Scripts](#20-reference-commands--scripts)
21. [Appendix](#21-appendix)

---

## 1. Executive Summary

This plan provides a structured, phased approach for migrating from **Azure DevOps Cloud** to **GitHub Enterprise Cloud** with **Enterprise Managed Users (EMU)**. The migration covers:

- **Source code repositories** (Git history, branches, tags, PRs)
- **CI/CD pipelines** (Azure Pipelines → GitHub Actions)
- **Work tracking** (Azure Boards → GitHub Issues/Projects or continued Azure Boards integration)
- **Packages/artifacts** (Azure Artifacts → GitHub Packages or registry migration)
- **Identity & access** (Azure AD/Entra ID → EMU SCIM provisioning)
- **Security scanning** (transition to GitHub Advanced Security)

### Why EMU?

Enterprise Managed Users (EMU) provides:
- **Centralized identity management**: User lifecycle controlled entirely from your IdP (Entra ID, Okta, PingFederate)
- **SCIM provisioning**: Automatic user creation, updates, and deactivation
- **OIDC/SAML SSO**: Users authenticate through your IdP
- **Conditional Access Policy (CAP)** support (with OIDC)
- **No public content**: Managed users cannot create public repositories — ideal for enterprises requiring strict data governance
- **IdP-controlled usernames, profiles, org membership, and repo access**

### Key Constraints of EMU

| Constraint | Impact |
|---|---|
| Users cannot create public repos or public content | All repos are private or internal |
| Users cannot collaborate outside the enterprise | No cross-enterprise forks or contributions |
| Cannot mix personal GitHub accounts | Managed accounts are separate from personal accounts |
| Username format is `IDP_SHORTCODE_USERNAME` | Usernames are auto-generated from IdP attributes |
| Only one email per user (from IdP) | Changing email in IdP unlinks contribution history |

---

## 2. Migration Scope & What Moves

### Data Migrated by GitHub Enterprise Importer (GEI)

| Data Type | Migrated? | Notes |
|---|---|---|
| Git source code (commits, branches, tags) | ✅ Yes | Full commit history preserved |
| Pull requests | ✅ Yes | Including comments, reviews, statuses |
| User history for pull requests | ✅ Yes | Attributed to mannequins initially |
| Work item links on pull requests | ✅ Yes | AB# links preserved |
| Attachments on pull requests | ✅ Yes | |
| Branch policies (repo-scoped) | ✅ Yes | User-scoped and cross-repo policies excluded |
| Git LFS pointers | ✅ Yes | LFS objects themselves are NOT migrated |
| Azure Pipelines YAML definitions | ⚠️ Partial | Pipeline YAML files in repo migrate as code; execution history does not |
| Azure Boards work items | ❌ No | Stay in ADO or use Azure Boards + GitHub integration |
| Azure Artifacts/Packages | ❌ No | Must be migrated separately |
| Wiki content | ❌ No | Must be migrated manually |
| Test Plans | ❌ No | Must be migrated separately |
| Build/release history | ❌ No | Historical pipeline runs stay in ADO |
| Dashboards | ❌ No | Must be recreated |
| Service connections | ❌ No | Must be reconfigured |
| Variable groups/secrets | ❌ No | Must be recreated as GitHub Secrets/Variables |

### Size Limits

| Limit | Value |
|---|---|
| Single Git commit | 2 GiB max |
| Git reference name | 255 bytes max |
| Single file in repo | 100 MiB (400 MiB during migration) |
| Git repository total blob size | 40 GiB (public preview) |
| Git reference name bytes | 255 bytes |

---

## 3. Phase 0 — Pre-Migration Assessment & Planning

### 3.1 Stakeholder Alignment

- [ ] Identify executive sponsor
- [ ] Identify migration lead (team/individual)
- [ ] Identify all affected teams and team leads
- [ ] Communicate migration rationale and timeline
- [ ] Establish a migration communication channel (Teams/Slack)
- [ ] Define success criteria and acceptance criteria

### 3.2 Inventory & Discovery

Run the ADO inventory report to understand the full scope:

```bash
gh ado2gh inventory-report --ado-org YOUR_ADO_ORG
```

This produces CSV files including `repos.csv` with:
- Repository names, sizes, and PR counts
- Team project mappings
- Pipeline information

**Key discovery items:**
- [ ] Total number of ADO organizations
- [ ] Total number of team projects per organization
- [ ] Total number of repositories (active vs. archived)
- [ ] Total number of pull requests across all repos
- [ ] Repository sizes (use `git-sizer` for detailed analysis)
- [ ] Presence of Git LFS usage
- [ ] Presence of Git submodules (especially cross-repo)
- [ ] Number of Azure Pipelines (YAML vs. Classic)
- [ ] Number of Azure Boards work items
- [ ] Azure Artifacts feeds and package counts
- [ ] Wiki usage
- [ ] Service hooks and integrations
- [ ] Third-party extensions in use
- [ ] Service connections (Azure, Docker, NuGet, etc.)
- [ ] Variable groups and secrets count
- [ ] Branch policies per repository

### 3.3 Repository Health Check

For each repository, run:

```bash
# Check repo size and potential issues
git clone --mirror https://dev.azure.com/ORG/PROJECT/_git/REPO
cd REPO.git
git-sizer --verbose
```

Flag repositories that:
- Exceed 10 GiB total blob size (plan for cleanup or Git LFS)
- Have files > 100 MiB (must use Git LFS post-migration)
- Have commit messages with non-ASCII characters that could cause reference issues
- Use TFVC (requires conversion to Git first)

### 3.4 TFVC Repositories

If any repositories use TFVC (Team Foundation Version Control), they must be converted to Git before migration:

- [ ] Identify all TFVC repos
- [ ] Use `git-tfs` or Azure DevOps built-in import to convert TFVC → Git
- [ ] Validate conversion completeness
- [ ] Plan for any TFVC-specific workflows that need adaptation

### 3.5 Licensing Assessment

- [ ] Determine required GitHub Enterprise Cloud seat count
- [ ] Determine GitHub Actions minute requirements
- [ ] Determine GitHub Packages storage requirements
- [ ] Determine GitHub Advanced Security (GHAS) license needs
- [ ] Determine GitHub Copilot license needs
- [ ] Compare with current Azure DevOps licensing costs

### 3.6 Migration Batching Strategy

**Recommended approach: Migrate in batches organized by ADO organization.**

| Strategy | When to Use |
|---|---|
| Big Bang | < 50 repos, low complexity, short maintenance window acceptable |
| Phased by ADO Organization | Multiple ADO orgs, moderate complexity |
| Phased by Team/Project | Teams have different readiness levels |
| Phased by Criticality | Start with low-risk repos, graduate to critical ones |

- [ ] Define batch composition
- [ ] Define batch sequence and timeline
- [ ] Identify dependencies between batches
- [ ] Define rollback criteria for each batch

---

## 4. Phase 1 — Enterprise & EMU Setup

### 4.1 GitHub Enterprise Account Procurement

- [ ] Purchase GitHub Enterprise Cloud with EMU
- [ ] Receive enterprise short code from GitHub (e.g., `_acmecorp`)
- [ ] Setup account with your GitHub account manager
- [ ] Determine hosting: **github.com** vs **GHE.com** (data residency)

> **GHE.com (Data Residency):** If your enterprise requires data to reside in a specific geographic region, use GHE.com with a custom subdomain (e.g., `acmecorp.ghe.com`). The API base URL would be `https://api.acmecorp.ghe.com`.

### 4.2 EMU Enterprise Configuration

- [ ] Sign in as the **setup user** (username: `SHORT-CODE_admin`)
- [ ] Configure enterprise settings:
  - Enterprise name and URL
  - Default repository visibility (private/internal)
  - Repository creation permissions
  - Forking policies
  - GitHub Actions policies
  - GitHub Packages policies
  - GitHub Copilot policies
- [ ] Configure enterprise-level security policies:
  - Two-factor authentication requirements
  - SSH certificate authorities (if needed)
  - IP allow list (if needed)

### 4.3 Choose Authentication Protocol

| Protocol | Pros | Cons |
|---|---|---|
| **OIDC** (Recommended) | CAP support, token auto-invalidation | Only Entra ID supported |
| **SAML** | Broader IdP support (Entra ID, Okta, PingFederate) | No CAP support |

- [ ] Select OIDC or SAML based on your IdP
- [ ] Document the decision and rationale

---

## 5. Phase 2 — Identity & Access Management

### 5.1 IdP Configuration (Entra ID Example)

#### SCIM Provisioning Setup

1. [ ] In Entra ID, go to **Enterprise Applications**
2. [ ] Add the **GitHub Enterprise Managed User** application (or **GitHub Enterprise Managed User (OIDC)**)
3. [ ] Configure provisioning:
   - Provisioning Mode: **Automatic**
   - Tenant URL: `https://api.github.com/scim/v2/enterprises/YOUR_ENTERPRISE`
   - Secret Token: PAT from the setup user with `admin:enterprise` scope
4. [ ] Map attributes:
   - `userPrincipalName` → `userName`
   - `mail` → `emails[type eq "work"].value`
   - `givenName` → `name.givenName`
   - `surname` → `name.familyName`
5. [ ] Configure group provisioning for team mapping
6. [ ] Test provisioning with a pilot group
7. [ ] Enable provisioning

#### SSO Configuration (OIDC or SAML)

- [ ] Configure SSO in the GitHub Enterprise Managed User application on Entra ID
- [ ] Set the reply URL / ACS URL per GitHub documentation
- [ ] Test SSO with the setup user
- [ ] Assign users and groups in Entra ID
- [ ] Verify users are provisioned to GitHub

### 5.2 Group & Team Mapping

| Entra ID Group | GitHub Organization | GitHub Team | Access Level |
|---|---|---|---|
| `ADO-OrgA-Admins` | `org-a` | `org-a-admins` | Admin |
| `ADO-OrgA-Contributors` | `org-a` | `org-a-contributors` | Write |
| `ADO-OrgA-Readers` | `org-a` | `org-a-readers` | Read |
| ... | ... | ... | ... |

- [ ] Define Entra ID group → GitHub team mappings
- [ ] Create Entra ID groups if they don't exist
- [ ] Assign users to appropriate groups
- [ ] Configure IdP group → GitHub team synchronization
- [ ] Validate team membership after provisioning

### 5.3 Enterprise Roles

| Role | Assigned Via | Purpose |
|---|---|---|
| Enterprise Owner | IdP (enterprise owner role) | Full enterprise administration |
| Enterprise Member | IdP (default) | Standard access within enterprise |
| Guest Collaborator | IdP (guest role) | Limited access for external collaborators |
| Billing Manager | Manually or IdP | Billing management only |

### 5.4 Username Considerations

EMU usernames are auto-generated: `IDP-SHORTCODE_IDP-USERNAME`

Example: If your enterprise shortcode is `acme` and IDP username is `jsmith`, the GitHub username will be `acme_jsmith`.

- [ ] Communicate new username format to all users
- [ ] Document how to find your GitHub username
- [ ] Update any automation that references usernames

---

## 6. Phase 3 — Tool Installation & Configuration

### 6.1 Install GitHub CLI & ADO2GH Extension

```bash
# Install GitHub CLI (macOS)
brew install gh

# Verify version >= 2.4.0
gh --version

# Install the ADO2GH extension
gh extension install github/gh-ado2gh

# Update to latest version
gh extension upgrade github/gh-ado2gh
```

### 6.2 Set Environment Variables

```bash
# Terminal (macOS/Linux)
export GH_PAT="ghp_your_github_pat_here"
export ADO_PAT="your_ado_pat_here"

# PowerShell (Windows)
$env:GH_PAT="ghp_your_github_pat_here"
$env:ADO_PAT="your_ado_pat_here"

# If using GHE.com (data residency)
export TARGET_API_URL="https://api.YOUR_SUBDOMAIN.ghe.com"
```

### 6.3 Required PAT Scopes

#### GitHub PAT (Classic) — Required Scopes

| Task | Scopes |
|---|---|
| Assign migrator role | `admin:org` |
| Run repository migration (org owner) | `repo`, `admin:org`, `workflow` |
| Run repository migration (migrator role) | `repo`, `read:org`, `workflow` |
| Download migration log | `repo`, `admin:org`, `workflow` |
| Reclaim mannequins | `admin:org` |

> **Note:** Fine-grained PATs are NOT supported for GEI. You must use classic PATs.

#### Azure DevOps PAT — Required Scopes

| Scope | Required For |
|---|---|
| `Work Items (Read)` | Reading work item links |
| `Code (Read)` | Reading source code and PRs |
| `Identity (Read)` | Reading user identities |
| `Full access` | Recommended for `inventory-report` |

- [ ] Allow the ADO PAT to access **all accessible organizations** if migrating from multiple orgs

### 6.4 Network Configuration

#### IP Allow List (if used)

Add these GitHub IP ranges to your allow lists:

```
192.30.252.0/22
185.199.108.0/22
140.82.112.0/20
143.55.64.0/20
135.234.59.224/28
2a0a:a440::/29
2606:50c0::/32
20.99.172.64/28
```

- [ ] Add GitHub IP ranges to organization IP allow list
- [ ] Temporarily disable IdP CAP restrictions during migration (if using OIDC)
- [ ] Add "Repository migrations" to bypass list for any repository rulesets

---

## 7. Phase 4 — Organizational Structure Design

### 7.1 Structural Mapping

**Azure DevOps structure:**
```
ADO Organization
  └── Team Project
        └── Repositories
        └── Pipelines
        └── Boards
        └── Artifacts
```

**GitHub Enterprise Cloud structure:**
```
Enterprise Account
  └── Organization
        ├── Teams
        ├── Repositories
        └── GitHub Packages
```

### 7.2 Mapping Recommendations

| ADO Concept | GitHub Equivalent | Notes |
|---|---|---|
| ADO Organization | GitHub Organization | 1:1 mapping recommended |
| Team Project | **No equivalent** — use Teams | Do NOT create 1 org per team project |
| Repository | Repository | 1:1 mapping |
| Team | Team | Map via IdP groups |
| Area paths | Labels or Projects | |
| Iteration paths | Milestones or Projects | |

### 7.3 Organization Naming Convention

```
enterprise: acme-corp
  ├── org: acme-platform       (from ADO org "Platform")
  ├── org: acme-product        (from ADO org "Product")
  ├── org: acme-infrastructure (from ADO org "Infrastructure")
  └── org: acme-shared         (for shared libraries/tools)
```

- [ ] Define organization naming convention
- [ ] Create organizations on GitHub Enterprise
- [ ] Configure organization settings (member privileges, default repo permissions, etc.)
- [ ] Create teams within organizations based on IdP group mapping
- [ ] Link teams to IdP groups for automatic membership sync

### 7.4 Repository Naming Convention

Consider standardizing repository names during migration:

| ADO Pattern | GitHub Pattern | Example |
|---|---|---|
| `ProjectName/RepoName` | `repo-name` | `platform-api` |
| Mixed case | lowercase-kebab | `MyService` → `my-service` |

- [ ] Define repository naming convention
- [ ] Document the ADO-to-GitHub name mapping
- [ ] Update the migration script to use `--target-repo` flag for renames

---

## 8. Phase 5 — Repository Migration (Trial Runs)

### 8.1 Generate Migration Script

```bash
gh ado2gh generate-script \
  --ado-org SOURCE_ADO_ORG \
  --github-org DESTINATION_GITHUB_ORG \
  --output migrate.ps1 \
  --all \
  --download-migration-logs

# If using GHE.com
gh ado2gh generate-script \
  --ado-org SOURCE_ADO_ORG \
  --github-org DESTINATION_GITHUB_ORG \
  --output migrate.ps1 \
  --all \
  --download-migration-logs \
  --target-api-url "$TARGET_API_URL"
```

The `--all` flag adds:
- Pipeline rewiring
- Team creation
- Azure Boards integration configuration

### 8.2 Review & Customize the Script

- [ ] Review generated `migrate.ps1`
- [ ] Remove or comment out repos you don't want to migrate
- [ ] Update `--target-repo` values for any renames
- [ ] Set `--target-repo-visibility` (default: matches source; options: `private`, `internal`)
- [ ] Verify the batch size is manageable

### 8.3 Create Trial Organization

```bash
# Create a sandbox organization for trial runs
# Naming convention: destination-org-sandbox
```

- [ ] Create sandbox/trial organizations (e.g., `acme-platform-sandbox`)
- [ ] Grant migrator role access to the sandbox org

### 8.4 Run Trial Migration

```bash
# Modify script to target sandbox org, then run
./migrate.ps1
```

### 8.5 Trial Validation Checklist

| Validation Item | Status |
|---|---|
| All expected repos exist in target org | ☐ |
| Git history is complete (commit count matches) | ☐ |
| All branches migrated correctly | ☐ |
| All tags migrated correctly | ☐ |
| Pull requests migrated with comments | ☐ |
| PR review comments preserved | ☐ |
| Work item links (AB#) preserved | ☐ |
| PR attachments accessible | ☐ |
| Branch policies migrated | ☐ |
| Large files handled correctly | ☐ |
| Migration logs reviewed for warnings | ☐ |
| Teams created correctly | ☐ |
| Repository visibility is correct | ☐ |
| No ruleset violations | ☐ |
| Git clone/push/pull works | ☐ |
| Code search works (after re-indexing) | ☐ |

### 8.6 Performance Benchmarking

- [ ] Record migration duration per repository
- [ ] Identify slow repositories (typically those with many PRs)
- [ ] Calculate total estimated production migration time
- [ ] Plan maintenance window accordingly

### 8.7 Resolve Trial Issues

- [ ] Document all issues found during trial
- [ ] Create remediation plan for each issue
- [ ] Re-run trial if significant changes were made
- [ ] Get sign-off from stakeholders on trial results

---

## 9. Phase 6 — Repository Migration (Production)

### 9.1 Pre-Migration Checklist

- [ ] All trial runs completed successfully
- [ ] Stakeholder sign-off obtained
- [ ] Communication sent to all affected teams
- [ ] Maintenance window scheduled
- [ ] ADO repos set to read-only (recommended)
- [ ] Rollback plan documented and tested
- [ ] On-call team identified for migration support

### 9.2 Halt Work Policy

> **CRITICAL:** GitHub Enterprise Importer does NOT support delta/incremental migrations. Any changes made to ADO repos after migration starts must be manually migrated.

**Options:**
1. **Full freeze** — Lock ADO repos before migration (recommended)
2. **Soft freeze** — Allow emergency changes only; manually reconcile after
3. **No freeze** — Accept that late changes require manual migration

### 9.3 Execute Production Migration

```bash
# Run the production migration script
./migrate.ps1
```

For large migrations, consider parallelizing:

```bash
# Single repo migration command
gh ado2gh migrate-repo \
  --ado-org SOURCE_ORG \
  --ado-team-project PROJECT \
  --ado-repo REPO_NAME \
  --github-org DEST_ORG \
  --target-repo TARGET_REPO_NAME \
  --target-repo-visibility internal

# If using GHE.com
gh ado2gh migrate-repo \
  --ado-org SOURCE_ORG \
  --ado-team-project PROJECT \
  --ado-repo REPO_NAME \
  --github-org DEST_ORG \
  --target-repo TARGET_REPO_NAME \
  --target-repo-visibility internal \
  --target-api-url "$TARGET_API_URL"
```

### 9.4 Monitor Migration Progress

```bash
# Check migration status
# Successful output:
# Migration completed (ID: RM_123)! State: SUCCEEDED

# If using --queue-only, check status with:
gh ado2gh wait-for-migration --migration-id RM_123
```

### 9.5 Post-Migration Repository Tasks

For each migrated repository:

- [ ] Verify migration status (SUCCEEDED)
- [ ] Review migration log (auto-created as an Issue in the repo)
- [ ] Set repository visibility (`private` or `internal`)
- [ ] Configure branch protection rules / rulesets
- [ ] Set up CODEOWNERS file
- [ ] Configure required status checks
- [ ] Push any Git LFS objects that weren't migrated
- [ ] Update `.gitmodules` if using submodules (new URLs)

### 9.6 Git LFS Migration

GEI migrates LFS pointers but NOT the LFS objects themselves:

```bash
# Clone the migrated repo from GitHub
git clone https://github.com/ORG/REPO.git
cd REPO

# Add ADO as a remote to fetch LFS objects
git remote add ado https://dev.azure.com/ORG/PROJECT/_git/REPO

# Fetch LFS objects from ADO
git lfs fetch ado --all

# Push LFS objects to GitHub
git lfs push origin --all
```

### 9.7 Reclaim Mannequins

After migration, all user activity (except Git commits) is attributed to placeholder identities called **mannequins**.

```bash
# Generate mannequin mapping CSV
gh ado2gh create-mannequin-csv --github-org YOUR_ORG

# Edit the CSV to map mannequins to real GitHub users

# Reclaim mannequins in bulk
gh ado2gh reclaim-mannequin --csv MANNEQUIN_MAP.csv --github-org YOUR_ORG
```

**Requirements:**
- Only organization owners can reclaim mannequins
- The target user must be a member of the organization
- For EMU, the target user must be a provisioned managed user

---

## 10. Phase 7 — CI/CD Pipeline Migration

### 10.1 Azure Pipelines → GitHub Actions Strategy

| Approach | When to Use |
|---|---|
| **Keep Azure Pipelines** (point to GitHub repos) | Short-term hybrid; minimal disruption |
| **Migrate to GitHub Actions** | Full platform consolidation; best long-term |
| **Use GitHub Actions Importer** | Automated conversion of pipeline definitions |
| **Manual rewrite** | Complex pipelines with custom tasks |

### 10.2 GitHub Actions Importer (Recommended First Step)

```bash
# Install the Actions Importer CLI extension
gh extension install github/gh-actions-importer

# Run audit to assess pipelines
gh actions-importer audit azure-devops \
  --output-dir audit-results \
  --azure-devops-organization SOURCE_ORG \
  --azure-devops-project PROJECT

# Dry-run conversion
gh actions-importer dry-run azure-devops \
  --output-dir dry-run-results \
  --azure-devops-organization SOURCE_ORG \
  --azure-devops-project PROJECT \
  --pipeline-id PIPELINE_ID

# Convert pipeline
gh actions-importer migrate azure-devops \
  --output-dir migration-results \
  --azure-devops-organization SOURCE_ORG \
  --azure-devops-project PROJECT \
  --pipeline-id PIPELINE_ID \
  --target-url https://github.com/DEST_ORG/DEST_REPO
```

### 10.3 Key Syntax Differences

| Azure Pipelines | GitHub Actions |
|---|---|
| `pool: vmImage: 'ubuntu-latest'` | `runs-on: ubuntu-latest` |
| `dependsOn: jobName` | `needs: jobName` |
| `condition: succeeded()` | `if: success()` |
| `task: TaskName@Version` | `uses: owner/action@version` |
| `script: command` | `run: command` |
| `variables:` | `env:` or GitHub Secrets |
| `stages:` | Separate workflow files or jobs |
| `template:` references | Reusable workflows (`workflow_call`) |
| `resources: repositories` | `actions/checkout` with `repository` param |
| Variable groups | GitHub Environments & Secrets |
| Service connections | GitHub Secrets + OIDC |
| Classic release pipelines | GitHub Actions with environments |
| Artifact publishing | `actions/upload-artifact` / `actions/download-artifact` |

### 10.4 Self-Hosted Runners

If you use Azure DevOps self-hosted agents:

| ADO Concept | GitHub Equivalent |
|---|---|
| Agent pools | Runner groups |
| Agent capabilities | Runner labels |
| Self-hosted agents | Self-hosted runners |
| Microsoft-hosted agents | GitHub-hosted runners |
| VMSS agents | GitHub Actions Runner Scale Sets (ARC) |

- [ ] Inventory existing self-hosted agents
- [ ] Plan runner infrastructure (VMs, Kubernetes with ARC, etc.)
- [ ] Install and register GitHub Actions runners
- [ ] Apply labels matching your workflow needs
- [ ] Configure runner groups for organization-level management
- [ ] Consider **Actions Runner Controller (ARC)** for Kubernetes-based autoscaling

### 10.5 Secrets & Variables Migration

```bash
# Set organization-level secrets
gh secret set SECRET_NAME --org ORG_NAME --body "secret_value"

# Set repository-level secrets
gh secret set SECRET_NAME --repo ORG/REPO --body "secret_value"

# Set environment secrets
gh secret set SECRET_NAME --env ENVIRONMENT --repo ORG/REPO --body "secret_value"
```

- [ ] Inventory all ADO variable groups and secrets
- [ ] Map to GitHub Secrets (org/repo/environment level)
- [ ] Recreate service connections as GitHub OIDC or secret-based auth
- [ ] Set up GitHub Environments for deployment gates/approvals
- [ ] Validate all secrets are correctly referenced in new workflows

### 10.6 Azure Pipelines Hybrid (Interim)

If you keep Azure Pipelines temporarily while pointing them at GitHub repos:

- [ ] Configure Azure Pipelines GitHub service connection
- [ ] Update pipeline triggers to use GitHub webhook events
- [ ] Test pipeline execution against GitHub repos
- [ ] Plan timeline for full Actions migration

---

## 11. Phase 8 — Work Item & Project Management Migration

### 11.1 Strategy Options

| Option | Pros | Cons |
|---|---|---|
| **Keep Azure Boards + GitHub integration** | Zero migration effort; mature PM tooling | Dual-platform; additional cost |
| **Migrate to GitHub Issues + Projects** | Single platform; simpler | Feature differences; migration effort |
| **Migrate to third-party tool** (Jira, etc.) | Best-of-breed PM | Additional licensing; migration complexity |

### 11.2 Option A: Keep Azure Boards (Recommended for Large Orgs)

Azure Boards integrates natively with GitHub repositories:

- [ ] Install the [Azure Boards app for GitHub](https://github.com/marketplace/azure-boards)
- [ ] Connect Azure Boards organizations to GitHub organizations
- [ ] Configure `AB#` link syntax in commit messages and PRs
- [ ] Verify work item links work bidirectionally
- [ ] Train developers on the `AB#1234` syntax for linking

### 11.3 Option B: Migrate to GitHub Issues + Projects

For organizations that want full consolidation:

**Manual migration considerations:**
- Work items → GitHub Issues
- Area paths → Labels
- Iteration paths → Milestones
- Custom fields → Custom fields in GitHub Projects
- Boards/Views → GitHub Projects views
- Queries → GitHub Issue search / saved filters

**Tools & approaches:**
- Azure DevOps REST API → GitHub REST/GraphQL API custom scripts
- Third-party tools: `azure-devops-to-github-issues` community scripts
- GitHub Projects (V2) for Kanban/Scrum boards

- [ ] Decide on migration approach
- [ ] Map work item types to GitHub Issue types
- [ ] Map custom fields to labels/projects fields
- [ ] Create migration script or use tooling
- [ ] Validate migrated work items
- [ ] Redirect references from old work items to new issues

---

## 12. Phase 9 — Packages & Artifacts Migration

### 12.1 Package Registry Mapping

| Azure Artifacts Feed Type | GitHub Packages Equivalent |
|---|---|
| NuGet | GitHub NuGet registry |
| npm | GitHub npm registry |
| Maven | GitHub Maven registry (Apache Maven) |
| Python (PyPI) | Not natively supported — use external PyPI |
| Universal Packages | Not supported — use GitHub Releases or external storage |

### 12.2 Migration Steps

- [ ] Inventory all Azure Artifacts feeds
- [ ] For each feed, identify consumers (pipelines, developers)
- [ ] Set up GitHub Packages registries
- [ ] Migrate packages (re-publish to GitHub Packages)
- [ ] Update client configuration files:
  - `nuget.config` — update registry URLs and credentials
  - `.npmrc` — update registry scope and auth
  - `settings.xml` (Maven) — update repository URLs
  - `pip.conf` / `pyproject.toml` — update index URLs
- [ ] Update CI/CD pipelines to publish to GitHub Packages
- [ ] Optionally set up upstream proxying
- [ ] Test package restore/install from new registries
- [ ] Deprecate Azure Artifacts feeds

### 12.3 GitHub Packages Authentication

```bash
# npm authentication (using PAT)
echo "//npm.pkg.github.com/:_authToken=TOKEN" >> ~/.npmrc

# NuGet authentication
dotnet nuget add source https://nuget.pkg.github.com/ORG/index.json \
  --name github --username USERNAME --password TOKEN

# Maven authentication (settings.xml)
# Add server entry with GitHub PAT
```

---

## 13. Phase 10 — Security & Compliance (GHAS)

### 13.1 GitHub Advanced Security (GHAS)

| Feature | Description |
|---|---|
| **Code Scanning** | SAST using CodeQL and third-party tools |
| **Secret Scanning** | Detect leaked secrets in repos |
| **Secret Scanning Push Protection** | Block pushes containing secrets |
| **Dependency Review** | Identify vulnerable dependencies in PRs |
| **Dependabot** | Automated dependency updates and security alerts |
| **Security Overview** | Enterprise-wide security dashboard |

### 13.2 GHAS Enablement Plan

- [ ] Enable GHAS at enterprise level
- [ ] Configure default setup for code scanning (CodeQL)
- [ ] Enable secret scanning for all organizations
- [ ] Enable push protection for secret scanning
- [ ] Configure Dependabot security updates
- [ ] Configure Dependabot version updates (`dependabot.yml`)
- [ ] Set up security policies (`SECURITY.md`)
- [ ] Configure security advisories workflow
- [ ] Review Security Overview dashboard

### 13.3 Code Scanning Configuration

```yaml
# .github/workflows/codeql-analysis.yml
name: "CodeQL"
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'  # Weekly Monday 6am

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write
    strategy:
      matrix:
        language: ['javascript', 'python', 'csharp']
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - name: Build (if compiled language)
        uses: github/codeql-action/autobuild@v3
      - name: Perform Analysis
        uses: github/codeql-action/analyze@v3
```

### 13.4 Dependabot Configuration

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
  - package-ecosystem: "nuget"
    directory: "/"
    schedule:
      interval: "weekly"
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

### 13.5 Compliance & Governance

- [ ] Configure repository rulesets at org/enterprise level
- [ ] Require signed commits (if applicable)
- [ ] Configure audit log streaming to SIEM
- [ ] Set up custom repository properties for compliance tracking
- [ ] Configure required workflows
- [ ] Review and configure GitHub Actions permissions (allowed actions list)

---

## 14. Phase 11 — Post-Migration Tasks & Validation

### 14.1 Immediate Post-Migration Checklist

| Task | Owner | Status |
|---|---|---|
| Verify migration status for all repos | Migration lead | ☐ |
| Review all migration logs | Migration lead | ☐ |
| Set repository visibility (private/internal) | Org owners | ☐ |
| Reclaim mannequins | Org owners | ☐ |
| Remove GEI IP ranges from allow lists | Platform team | ☐ |
| Re-enable IdP CAP restrictions | Identity team | ☐ |
| Remove "Repository migrations" from ruleset bypass | Platform team | ☐ |
| Push Git LFS objects | Dev teams | ☐ |
| Update Git remote URLs on developer machines | Developers | ☐ |
| Test clone/push/pull for all repos | Dev teams | ☐ |
| Verify branch protection rules | Org owners | ☐ |
| Verify team access and permissions | Org owners | ☐ |
| Test CI/CD pipelines | Dev teams | ☐ |
| Verify package feeds | Dev teams | ☐ |
| Run initial code scanning | Security team | ☐ |
| Verify secret scanning results | Security team | ☐ |

### 14.2 Developer Workstation Updates

Each developer must:

1. **Update Git remotes:**
   ```bash
   cd my-repo
   git remote set-url origin https://github.com/ORG/REPO.git
   ```

2. **Update authentication:**
   - For HTTPS: Configure Git credential manager with GitHub credentials
   - For SSH: Add SSH key to GitHub managed user account

3. **Update IDE integrations:**
   - VS Code: Sign in with EMU account
   - Visual Studio: Update GitHub connection
   - JetBrains: Update GitHub plugin settings

4. **Update local tool configurations:**
   - `.npmrc`, `nuget.config`, etc. for package registries
   - Docker login for container registries

### 14.3 Integration Updates

- [ ] Update CI/CD service connections
- [ ] Update webhook consumers
- [ ] Update ChatOps integrations (Slack, Teams)
- [ ] Update monitoring and alerting (PagerDuty, OpsGenie, etc.)
- [ ] Update documentation references
- [ ] Update internal wikis and runbooks
- [ ] Update any scripts that reference ADO repository URLs

### 14.4 Smoke Testing

For each migrated repository:
- [ ] Clone the repository
- [ ] Create a feature branch
- [ ] Make a change, commit, and push
- [ ] Create a pull request
- [ ] Verify CI/CD runs on the PR
- [ ] Merge the pull request
- [ ] Verify deployment pipeline triggers (if applicable)

---

## 15. Phase 12 — Developer Onboarding & Training

### 15.1 Training Topics

| Topic | Audience | Format |
|---|---|---|
| GitHub Fundamentals (UI, PR flow, Issues) | All developers | Self-paced / Workshop |
| EMU Authentication & Account | All users | Documentation + FAQ |
| GitHub Actions vs Azure Pipelines | CI/CD engineers | Workshop |
| GitHub Advanced Security | Security team | Workshop |
| GitHub Copilot | All developers | Self-paced / Demo |
| GitHub Projects | Project managers | Workshop |
| GitHub API & Automation | Platform team | Workshop |
| CODEOWNERS & Branch Protection | Team leads | Documentation |
| GitHub Packages | Package maintainers | Documentation |

### 15.2 Key Workflow Differences to Communicate

| Azure DevOps | GitHub |
|---|---|
| Work Items | Issues |
| Boards | Projects |
| Branch Policy auto-created reviewers | CODEOWNERS file |
| Classic editor pipelines | YAML only (GitHub Actions) |
| Variable groups | Organization/repository/environment secrets |
| Service connections | OIDC federation or secret-based auth |
| Approved/Required reviewers in branch policy | Branch protection rules + CODEOWNERS |
| `dependsOn` for pipeline job ordering | `needs` for job dependencies |
| `condition:` for conditional execution | `if:` for conditional execution |
| Build artifacts | Upload/download artifact actions |
| Release pipelines with stages | GitHub Actions with environments |
| Agent pools | Runner groups |

### 15.3 Developer Resources

- [ ] Create internal migration FAQ
- [ ] Create "Getting Started with GitHub" guide (EMU-specific)
- [ ] Set up a `#github-migration-support` channel
- [ ] Schedule office hours for migration questions
- [ ] Create cheat sheet: ADO → GitHub command/concept mapping
- [ ] Document Git remote URL update instructions
- [ ] Create CODEOWNERS templates for teams
- [ ] Document GitHub Actions best practices for your org

---

## 16. Phase 13 — Decommission Azure DevOps

### 16.1 Coexistence Period

**Recommended minimum coexistence: 30-90 days post-migration**

During this period:
- [ ] Keep ADO repos in **read-only mode**
- [ ] Monitor for any references to ADO repos
- [ ] Ensure all integrations are pointing to GitHub
- [ ] Verify no active builds running against ADO repos
- [ ] Keep Azure Boards accessible (if still in use)

### 16.2 Decommission Steps

1. [ ] Confirm all repos have been successfully migrated and validated
2. [ ] Confirm all pipelines are running on GitHub Actions (or still using Azure Pipelines + GitHub)
3. [ ] Confirm all packages are available from new registries
4. [ ] Archive ADO organizations (set to read-only)
5. [ ] Export any remaining data (wiki, test plans, dashboards)
6. [ ] Revoke ADO PATs created for migration
7. [ ] Remove ADO service connections that are no longer needed
8. [ ] Update DNS/redirects if applicable
9. [ ] Cancel/downgrade Azure DevOps licenses
10. [ ] Document the final state and archive migration records

### 16.3 Data Retention

- [ ] Determine data retention requirements for ADO data
- [ ] Export audit logs from Azure DevOps
- [ ] Archive ADO organization (do not delete immediately)
- [ ] Set a calendar reminder for final deletion (e.g., 1 year post-migration)

---

## 17. Risk Register & Mitigations

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| 1 | Large repos fail migration (>40 GiB) | Medium | High | Pre-screen with `git-sizer`; clean up history; split repos |
| 2 | Data loss during migration | Low | Critical | Run trial migrations; validate checksums; keep ADO read-only |
| 3 | Pipeline conversion failures | High | High | Use Actions Importer for audit; plan manual conversion time |
| 4 | Developer productivity dip | Medium | Medium | Training, documentation, support channels, office hours |
| 5 | EMU provisioning issues | Medium | High | Test with pilot group; validate SCIM before bulk provisioning |
| 6 | Mannequin reclaim failures | Low | Medium | Ensure all users are provisioned before reclaiming |
| 7 | Secret exposure during migration | Low | Critical | Review migration logs for leaked secrets; enable secret scanning |
| 8 | Build downtime | Medium | High | Plan maintenance window; keep ADO pipelines as fallback |
| 9 | Package registry disruption | Medium | High | Dual-publish during transition; test restores before cutover |
| 10 | Git LFS objects not migrated | High | Medium | Document LFS push process; automate with scripts |
| 11 | TFVC repos incompatible | Medium | High | Convert to Git in ADO first; validate before migration |
| 12 | Cross-repo submodule references break | Medium | Medium | Update `.gitmodules` paths; test before cutover |
| 13 | Rulesets block migration | Medium | Medium | Add "Repository migrations" to ruleset bypass lists |
| 14 | Code search unavailable temporarily | High | Low | Communicate expected re-indexing delay (hours) |
| 15 | Classic (non-YAML) pipelines can't be auto-converted | High | High | Manual rewrite required; prioritize by criticality |

---

## 18. RACI Matrix

| Activity | Executive Sponsor | Migration Lead | Platform/DevOps | Security | Dev Teams | IdP Admin |
|---|---|---|---|---|---|---|
| Migration decision & budget | **A** | R | C | C | I | I |
| Enterprise/EMU setup | I | **A** | R | C | I | C |
| IdP/SCIM configuration | I | C | C | C | I | **R/A** |
| Tool installation & config | I | **A** | R | I | I | I |
| Org structure design | C | **A** | R | C | C | I |
| Trial migration | I | **A** | R | I | C | I |
| Production migration | I | **A** | R | I | C | I |
| Pipeline migration | I | C | **R/A** | I | R | I |
| Package migration | I | C | **R/A** | I | R | I |
| GHAS enablement | I | C | R | **A** | I | I |
| Mannequin reclaims | I | **A** | R | I | C | I |
| Developer onboarding | I | **A** | R | I | R | I |
| ADO decommission | **A** | R | R | C | I | C |

**R** = Responsible, **A** = Accountable, **C** = Consulted, **I** = Informed

---

## 19. Key Differences: Azure DevOps vs GitHub

### Structural Differences

| Feature | Azure DevOps | GitHub |
|---|---|---|
| Hierarchy | Org → Project → Repo | Enterprise → Org → Repo |
| Grouping concept | Team Projects | Teams + Topics |
| Code review | Pull Requests | Pull Requests |
| CI/CD | Azure Pipelines (YAML + Classic) | GitHub Actions (YAML only) |
| Work tracking | Azure Boards | GitHub Issues + Projects |
| Package management | Azure Artifacts | GitHub Packages |
| Wiki | Azure Wiki | GitHub Wiki or repo-based docs |
| Test management | Azure Test Plans | No direct equivalent |
| Dashboards | Azure Dashboards | GitHub Insights + Actions |
| Security scanning | Azure DevOps extensions | GitHub Advanced Security |

### Authentication Differences

| Feature | Azure DevOps | GitHub EMU |
|---|---|---|
| User accounts | AAD-linked personal accounts | Managed user accounts (IdP-controlled) |
| SSO | Entra ID SSO | SAML or OIDC SSO |
| PATs | Azure DevOps PATs | GitHub PATs (classic or fine-grained) |
| SSH keys | Supported | Supported |
| Service principals | Service connections | GitHub Apps / OIDC federation |

---

## 20. Reference Commands & Scripts

### Inventory Report

```bash
gh ado2gh inventory-report --ado-org YOUR_ADO_ORG
```

### Generate Migration Script

```bash
gh ado2gh generate-script \
  --ado-org SOURCE_ORG \
  --github-org DEST_ORG \
  --output migrate.ps1 \
  --all \
  --download-migration-logs
```

### Single Repo Migration

```bash
gh ado2gh migrate-repo \
  --ado-org SOURCE_ORG \
  --ado-team-project PROJECT \
  --ado-repo REPO \
  --github-org DEST_ORG \
  --target-repo TARGET_REPO \
  --target-repo-visibility internal
```

### Check Migration Status

```bash
gh ado2gh wait-for-migration --migration-id RM_XXXXX
```

### Reclaim Mannequins

```bash
gh ado2gh create-mannequin-csv --github-org YOUR_ORG
# Edit the CSV, then:
gh ado2gh reclaim-mannequin --csv mapping.csv --github-org YOUR_ORG
```

### Set Repository Visibility in Bulk

```bash
export ORG=YOUR_ORG
gh repo list "$ORG" --limit 100000 --json name -q '.[].name' | \
  xargs -I{} gh repo edit "$ORG/{}" --visibility internal
```

### Actions Importer Audit

```bash
gh actions-importer audit azure-devops \
  --output-dir audit-results \
  --azure-devops-organization SOURCE_ORG \
  --azure-devops-project PROJECT
```

### Actions Importer Migration

```bash
gh actions-importer migrate azure-devops \
  --output-dir migration-results \
  --azure-devops-organization SOURCE_ORG \
  --azure-devops-project PROJECT \
  --pipeline-id PIPELINE_ID \
  --target-url https://github.com/DEST_ORG/DEST_REPO
```

---

## 21. Appendix

### A. Glossary

| Term | Definition |
|---|---|
| **GEI** | GitHub Enterprise Importer — the migration tool |
| **ADO2GH** | GitHub CLI extension for Azure DevOps to GitHub migration |
| **EMU** | Enterprise Managed Users — GitHub accounts managed by your IdP |
| **SCIM** | System for Cross-domain Identity Management — user provisioning protocol |
| **OIDC** | OpenID Connect — authentication protocol (recommended for EMU with Entra ID) |
| **SAML** | Security Assertion Markup Language — authentication protocol |
| **CAP** | Conditional Access Policy — IdP-based access controls |
| **Mannequin** | Placeholder identity for migrated user activity before reclaiming |
| **GHE.com** | GitHub Enterprise with data residency (custom subdomain) |
| **GHAS** | GitHub Advanced Security |
| **ARC** | Actions Runner Controller — Kubernetes-based runner autoscaling |

### B. Official Documentation Links

| Resource | URL |
|---|---|
| GEI Overview | https://docs.github.com/en/migrations/using-github-enterprise-importer |
| ADO to GitHub Migration Guide | https://docs.github.com/en/migrations/ado |
| About EMU | https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/identity-and-access-management/enterprise-managed-users |
| Azure Pipelines to GitHub Actions | https://docs.github.com/en/actions/tutorials/migrate-to-github-actions/manual-migrations/migrate-from-azure-pipelines |
| GitHub Actions Importer | https://docs.github.com/en/actions/migrating-to-github-actions/using-github-actions-importer |
| GitHub Advanced Security | https://docs.github.com/en/code-security |
| Azure Boards + GitHub Integration | https://learn.microsoft.com/en-us/azure/devops/boards/github |
| Azure Pipelines + GitHub Integration | https://learn.microsoft.com/en-us/azure/devops/pipelines/repos/github |
| Key Differences ADO vs GitHub | https://docs.github.com/en/migrations/ado/key-differences-between-azure-devops-and-github |

### C. EMU Supported Identity Providers

| IdP | SAML Auth | OIDC Auth | SCIM Provisioning |
|---|---|---|---|
| Microsoft Entra ID | ✅ | ✅ | ✅ |
| Okta | ✅ | ❌ | ✅ |
| PingFederate | ✅ | ❌ | ✅ |
| Other SAML 2.0 + SCIM 2.0 | ✅ | ❌ | ✅ (not officially supported) |

> **Note:** Mixing Okta and Entra ID for SSO/SCIM (in either order) is explicitly NOT supported.

### D. Migration Timeline Template

| Week | Activities |
|---|---|
| Week 1-2 | Assessment, inventory, stakeholder alignment |
| Week 3-4 | Enterprise/EMU setup, IdP configuration, tool installation |
| Week 5-6 | Org structure design, team mapping, permissions planning |
| Week 7-8 | Trial migration (batch 1), validation, issue resolution |
| Week 9-10 | Trial migration (batch 2+), pipeline conversion pilot |
| Week 11-12 | Production migration (batch 1), mannequin reclaims |
| Week 13-14 | Production migration (batch 2+), pipeline migration |
| Week 15-16 | Package migration, GHAS enablement |
| Week 17-18 | Developer onboarding, training, smoke testing |
| Week 19-20 | Stabilization, issue resolution, Azure Boards integration |
| Week 21+   | Coexistence monitoring, ADO decommission planning |

> **Note:** Timeline varies significantly based on number of repos, PRs, pipelines, and organizational complexity. Adjust accordingly.

---

*This plan should be reviewed and customized for your specific organizational context. Engage your GitHub account team and consider GitHub Expert Services for complex migrations.*
