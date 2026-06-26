# React JS Deployment to Azure App Service via Azure DevOps CI/CD

This README documents the end-to-end process of deploying a **React JS** application to **Azure App Service** using an **Azure DevOps CI/CD pipeline**.

## Overview

The project (`React2606`) is built and deployed using the following flow:

1. Create an Azure App Service (Web App) to host the React frontend
2. Set up a Build pipeline (CI) in Azure DevOps to install dependencies and build the app
3. Publish the build output as an artifact
4. Set up a Release pipeline (CD) to deploy the artifact to Azure App Service
5. Verify the deployed app is live

---

## 1. Create the Azure Web App

In the Azure Portal, a new **App Service Web App** is created with the following configuration:

| Setting | Value |
|---|---|
| Subscription | Azure subscription 1 |
| Resource Group | ab123 |
| Name | React2606 |
| Publish | Code |
| Runtime stack | Node 22 LTS |
| Operating System | Windows |
| Region | Korea South |

This provisions the hosting environment at:
`react2606-dwcjbqaqa4fhcjbc.koreasouth-01.azurewebsites.net`

---

## 2. Connect the Repository in Azure DevOps

A new pipeline is created in Azure DevOps under the **React** project (Pipelines → New Pipeline), connecting to the source repository (Azure Repos Git, GitHub, Bitbucket Cloud, or other Git providers).

---

## 3. Build Pipeline (CI) — `React-CI`

The CI pipeline is configured with the following tasks, run on an **Agent Job 1**:

### Task 1: `npm install`
- **Task**: npm
- **Command**: `install`
- Installs all project dependencies defined in `package.json`.

### Task 2: `npm run build`
- **Task**: npm
- **Command**: `custom`
- **Command and arguments**: `run build`
- Builds the production-ready React app (generates the `build` folder).

### Task 3: Publish Artifact: drop
- **Task**: Publish build artifacts
- **Path to publish**: `build`
- **Artifact name**: `drop`
- **Artifact publish location**: Azure Pipelines
- Publishes the compiled `build` folder as a pipeline artifact named `drop`, making it available to the release pipeline.

### Pipeline Run Result
Run **#108** of `React-CI (12)` completed successfully:

```
✔ Initialize job
✔ Checkout React/JS@main
✔ npm install            (56s)
✔ npm run build          (18s)
✔ Publish Artifact: drop (1s)
✔ Post-job: Checkout React
✔ Finalize Job
✔ Report build status
```

Total job duration: **1m 26s** — 1 artifact produced.

---

## 4. Release Pipeline (CD) — `New release pipeline`

A release pipeline is configured with a single stage (**Stage 1**) containing one task:

### Task: Azure App Service Deploy
- **Display name**: Azure App Service Deploy: React2606
- **Connection type**: Azure Resource Manager
- **Azure subscription**: Permanent
- **App Service type**: Web App on Windows
- **App Service name**: React2606
- **Package or folder**: `$(System.DefaultWorkingDirectory)/_React-CI (12)/drop`

This task takes the `drop` artifact produced by the CI build and deploys its contents directly to the `React2606` App Service.

### Release Run Result
**Release-1**, Stage 1, completed successfully:

```
✔ Initialize job
✔ Download artifact - _React-CI (12) - drop  (2s)
✔ Azure App Service Deploy: React2606         (1m 5s)
✔ Finalize Job                                (<1s)
```

Total deployment duration: **1m 18s**

---

## 5. Verify Deployment

In the Azure Portal, the **React2606** App Service overview confirms:

| Property | Value |
|---|---|
| Status | Running |
| Resource Group | ab123 |
| Location | Korea South |
| Runtime Stack | Node - 22 |
| Operating System | Windows |
| Default Domain | react2606-dwcjbqaqa4fhcjbc.koreasouth-01.azurewebsites.net |

Browsing to the app's default domain confirms the React app is live:

> **React JS Frontend 1**
> Navigation: `Home` | `Department` | `Employee`

---

## Summary Pipeline Flow

```
 ┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌──────────────────┐
 │  Source Repo │ --> │  CI: npm     │ --> │ Publish       │ --> │ CD: Deploy to     │
 │  (main)      │     │  install +   │     │ Artifact      │     │ Azure App Service │
 │              │     │  run build   │     │ ("drop")      │     │ (React2606)       │
 └─────────────┘     └──────────────┘     └───────────────┘     └──────────────────┘
                                                                          │
                                                                          v
                                                            react2606-dwcjbqaqa4fhcjbc.
                                                            koreasouth-01.azurewebsites.net
```

## Tech Stack

- **Frontend**: React JS
- **Runtime**: Node 22 LTS
- **Hosting**: Azure App Service (Windows)
- **CI/CD**: Azure DevOps Pipelines (Build + Classic Release)
- **Artifact Storage**: Azure Pipelines artifact (`drop`)

## Notes

- The App Service overview shows a prompt to enable **Application Insights** for monitoring and profiling — recommended for production visibility.
- The Health Check feature was unable to fetch data at the time of this snapshot; this can be revisited under **Diagnose and solve problems**.
