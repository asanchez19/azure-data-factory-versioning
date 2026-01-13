# Azure Data Factory Test Repository

This repository is used to test **Spike EO-365: Define Git-Based Development Workflow for Azure Data Factory and Data Lake**.

## Purpose

The goal of this spike is to establish a Git-based development workflow for Azure Data Factory (ADF) that enables:

- Source control and version management for ADF pipelines
- Collaboration through pull requests and code reviews
- CI/CD pipeline integration for automated deployments
- Environment promotion (Dev → QA → Prod)

## Proposed Workflow

### Repository Strategy

**Option A - Dedicated Github repo for ADF (Recommended)**
- ADF code lives only in Github

### Branching Strategy

| Branch Type | Naming Convention | Purpose |
|-------------|-------------------|---------|
| Collaboration | `main` | Main integration branch |
| Feature | `feature/<ticket>-<title>` | New feature development |
| Bugfix | `bugfix/<ticket>-<title>` | Bug fixes |

### Environments

| Environment | Git Connected | Description |
|-------------|---------------|-------------|
| `datafactory-dev` | Yes (Azure Repos/GitHub) | Developers build and test here |
| `datafactory-qa` | No | Receives ARM template deployments from publish branch |
| `datafactory-prod` | No | Receives ARM template deployments with manual approval |

## End-to-End Process Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                              ADF GIT-BASED DEVELOPMENT WORKFLOW                         │
└─────────────────────────────────────────────────────────────────────────────────────────┘

┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  1. DEVELOP  │────▶│  2. REVIEW   │────▶│  3. PUBLISH  │────▶│ 4. DEPLOY QA │────▶│ 5. PROD      │
│              │     │   & MERGE    │     │              │     │              │     │              │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
       │                    │                    │                    │                    │
       ▼                    ▼                    ▼                    ▼                    ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Create       │     │ Code review  │     │ Release Mgr  │     │ Pipeline     │     │ Manual       │
│ feature/     │     │ in Git       │     │ publishes    │     │ triggers on  │     │ approval     │
│ bugfix       │     │              │     │ from main    │     │ adf_publish  │     │ required     │
│ branch       │     │ PR approvals │     │              │     │ branch       │     │              │
│              │     │              │     │ Generates    │     │              │     │ Deploy to    │
│ Edit in ADF  │     │ Merge to     │     │ ARM templates│     │ Deploy to    │     │ datafactory- │
│ (Git mode)   │     │ main         │     │ in adf_      │     │ datafactory- │     │ prod         │
│              │     │              │     │ publish      │     │ qa           │     │              │
│ Commit &     │     │              │     │ branch       │     │              │     │              │
│ open PR      │     │              │     │              │     │              │     │              │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘


                              DETAILED FLOW DIAGRAM

    ┌─────────────────────────────────────────────────────────────────────────────────┐
    │                                  GIT REPOSITORY                                  │
    │  ┌─────────────────────────────────────────────────────────────────────────┐    │
    │  │                              main (collaboration)                        │    │
    │  └───────────────────────────────────▲─────────────────────────────────────┘    │
    │                                      │ PR Merge                                  │
    │  ┌───────────────────────────────────┴─────────────────────────────────────┐    │
    │  │                    feature/<ticket>-<title>                              │    │
    │  │                    bugfix/<ticket>-<title>                               │    │
    │  └─────────────────────────────────────────────────────────────────────────┘    │
    │                                                                                  │
    │  ┌─────────────────────────────────────────────────────────────────────────┐    │
    │  │                           adf_publish (auto-generated)                   │    │
    │  │                         Contains ARM templates                           │    │
    │  └─────────────────────────────────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────────────────────────────────┘
                                           │
                                           │ Publish generates ARM templates
                                           ▼
    ┌─────────────────────────────────────────────────────────────────────────────────┐
    │                                CI/CD PIPELINE                                    │
    │                                                                                  │
    │   ┌─────────────┐          ┌─────────────┐          ┌─────────────┐            │
    │   │             │  Auto    │             │  Manual  │             │            │
    │   │  DEV ADF    │─────────▶│   QA ADF    │─────────▶│  PROD ADF   │            │
    │   │             │  Deploy  │             │ Approval │             │            │
    │   │ (Git Mode)  │          │ (ARM Only)  │          │ (ARM Only)  │            │
    │   └─────────────┘          └─────────────┘          └─────────────┘            │
    │                                                                                  │
    └─────────────────────────────────────────────────────────────────────────────────┘
```

## Process Steps

### 1. Develop
- Developer creates a `feature/` or `bugfix/` branch from `main`
- Developer edits pipelines/datasets/linked services in ADF (Git mode)
- Developer commits changes to their branch and opens a PR → `main`

### 2. Review & Merge
- Code review happens in Git (comments, approvals, required checks)
- PR merges into `main` (collaboration branch)

### 3. Publish (Release Managers Only)
- A Release Manager performs ADF Publish from `main`
- Publishing deploys changes into the dev factory's live state
- Generates ARM templates into the publish branch (`adf_publish`)

> **Why restrict Publish?** Because Publish is the "gateway" that produces the deployable artifacts and can impact the live state of the dev factory.

### 4. Deploy to QA (Automatic)
- A pipeline triggers when the publish branch updates (new commit in `adf_publish`)
- The pipeline deploys the generated ARM templates to `datafactory-qa`

### 5. Promote to Production (With Approval)
- Promotion to `datafactory-prod` requires manual approval in the release pipeline
- After approval, the same published templates are deployed to prod

## Resources

- [Source control - Azure Data Factory](https://learn.microsoft.com/en-us/azure/data-factory/source-control)
- [Continuous integration and delivery - Azure Data Factory](https://learn.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery)
- [Templates overview - Azure Resource Manager](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/overview)

## Best Practices

- All team members should have read permissions to the Data Factory
- Only a select set of people should be allowed to publish to the Data Factory
- Do not allow direct check-ins to the collaboration branch
- Use `_` or `-` characters instead of spaces in resource names
- Configure only your development data factory with Git integration
- Use PowerShell scripts for pre- and post-deployment tasks
