# Azure DevOps → GitHub Enterprise Cloud (EMU) Migration

A comprehensive migration guide and operational checklist for migrating from **Azure DevOps Cloud** to **GitHub Enterprise Cloud** with **Enterprise Managed Users (EMU)**.

---

## Overview

This repository contains all the planning artifacts needed to execute a full migration from Azure DevOps to GitHub Enterprise Cloud (EMU), covering:

- **Source code repositories** — Git history, branches, tags, pull requests
- **CI/CD pipelines** — Azure Pipelines → GitHub Actions
- **Identity & access** — Entra ID SCIM provisioning for EMU
- **Work tracking** — Azure Boards integration or migration to GitHub Issues/Projects
- **Packages & artifacts** — Azure Artifacts → GitHub Packages
- **Security** — GitHub Advanced Security (GHAS) enablement

## Documents

| Document | Description |
|---|---|
| [ADO-to-GHEC-EMU-Migration-Plan.md](ADO-to-GHEC-EMU-Migration-Plan.md) | Full migration plan with 13 phases, technical details, reference commands, risk register, and RACI matrix |
| [Migration-Checklist.md](Migration-Checklist.md) | Actionable 299-item checklist with assignee/date tracking across all phases |

## Migration Phases

| Phase | Summary |
|---|---|
| **Phase 0** | Pre-migration assessment — inventory, TFVC detection, licensing, batching strategy |
| **Phase 1** | GitHub Enterprise & EMU account setup (OIDC vs SAML) |
| **Phase 2** | Identity & access — Entra ID SCIM provisioning, group/team mapping |
| **Phase 3** | Tool installation — `gh-ado2gh` CLI extension, PATs, network config |
| **Phase 4** | Organizational structure design — ADO org/project → GitHub org/team mapping |
| **Phase 5** | Trial migrations with validation checklist |
| **Phase 6** | Production repository migration, LFS handling, mannequin reclaims |
| **Phase 7** | CI/CD pipeline migration (Actions Importer, self-hosted runners, secrets) |
| **Phase 8** | Work items — Azure Boards integration or GitHub Issues/Projects migration |
| **Phase 9** | Package registry migration (NuGet, npm, Maven) |
| **Phase 10** | Security & compliance — GHAS, CodeQL, secret scanning, Dependabot |
| **Phase 11** | Post-migration validation, developer workstation updates, smoke testing |
| **Phase 12** | Developer onboarding & training |
| **Phase 13** | Azure DevOps decommission & data retention |

## Key Tools

| Tool | Purpose |
|---|---|
| [GitHub CLI](https://cli.github.com/) | Command-line interface for GitHub |
| [gh-ado2gh](https://github.com/github/gh-ado2gh) | GitHub CLI extension for ADO → GitHub repo migration |
| [GitHub Actions Importer](https://docs.github.com/en/actions/migrating-to-github-actions/using-github-actions-importer) | Automated Azure Pipelines → GitHub Actions conversion |
| [git-sizer](https://github.com/github/git-sizer) | Pre-migration repository size analysis |

## Quick Start

```bash
# 1. Install tools
brew install gh
gh extension install github/gh-ado2gh

# 2. Set credentials
export GH_PAT="ghp_your_github_pat"
export ADO_PAT="your_ado_pat"

# 3. Run inventory
gh ado2gh inventory-report --ado-org YOUR_ADO_ORG

# 4. Generate migration script
gh ado2gh generate-script \
  --ado-org SOURCE_ORG \
  --github-org DEST_ORG \
  --output migrate.ps1 \
  --all

# 5. Run trial migration, then production
./migrate.ps1
```

## What Gets Migrated (via GEI)

| Data | Migrated |
|---|---|
| Git source (commits, branches, tags) | ✅ |
| Pull requests (comments, reviews) | ✅ |
| Work item links on PRs | ✅ |
| PR attachments | ✅ |
| Branch policies (repo-scoped) | ✅ |
| Git LFS pointers | ✅ (objects require manual push) |
| Azure Pipelines (YAML files) | ⚠️ As code only |
| Azure Boards work items | ❌ |
| Azure Artifacts / Packages | ❌ |
| Wikis, Test Plans, Dashboards | ❌ |
| Build/release history | ❌ |

## EMU Requirements

- **Identity Provider:** Entra ID (OIDC + SCIM), Okta (SAML + SCIM), or PingFederate (SAML + SCIM)
- **Authentication:** OIDC recommended (Entra ID only) for Conditional Access Policy support
- **User accounts:** Fully managed by IdP — no personal GitHub accounts
- **Public content:** Not allowed — all repos are private or internal

## Contributing

To suggest improvements to these migration artifacts, open a pull request or issue in this repository.

## License

Internal use only. Please consult your organization's policies before sharing externally.
