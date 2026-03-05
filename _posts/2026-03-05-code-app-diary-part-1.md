---
layout: single
title: "Code App Diary · Part 1: From Blank Vite Shell to hopefully MVP application"
date: 2026-03-05 00:00:00 -0500
categories: [Power Platform]
tags: [Power Apps, Power Apps code components, PAC CLI, Vite, React]
toc: true
excerpt: "My first attempts at building a Power Apps code app from scratch: setting up the toolchain, fighting through prompting via Codex and GitHub Copilot"
---

## Kickstarting a Code App Environment

Before I could even touch JSX, I had to give my workstation all the toys a code app expects. If you are starting from scratch, these are the steps I wish someone had printed on a single page. I rewrote this section to match Microsoft’s official quickstart plus the exact steps I followed for the company project. It walks through the journey from pretty much clean laptop to first deployed build. I had "pac" installed previously, same goes for .NET but I ended up installing it again or in case of .NET updating it to the version 10.

### 1. Prerequisites

| Tool | Why | Install command |
| --- | --- | --- |
| Node.js 20 LTS | Required by Vite + React | `brew install node@20 && echo 'export PATH="/opt/homebrew/opt/node@20/bin:$PATH"' >> ~/.zprofile` (macOS) or `winget install OpenJS.NodeJS.LTS` (Windows) |
| PNPM/NPM | Package manager (NPM ships with Node) | n/a |
| .NET 8 SDK | Power Apps CLI dependency | `brew install dotnet-sdk` or `winget install Microsoft.DotNet.SDK.8` |
| Power Platform CLI (`pac`) | Auth, scaffolding, metadata, deployment | `brew install --cask powerplatform-cli` or `winget install Microsoft.PowerAppsCLI` |

Verify the installs:

```bash
node -v
npm -v
dotnet --version
pac --version
```

### 2. (Optional) Bootstrap a new template

If you are not using the sample repo, scaffold Microsoft’s Vite starter:

```bash
npx degit github:microsoft/PowerAppsCodeApps/templates/vite my-code-app
cd my-code-app
npm install
pac code init --displayname "My Code App"
```

### 3. Clone & install this repo

```bash
git clone https://github.com/<org>/<repo>.git
cd <repo>/invoicing-platform
npm install
```

### 4. Create an auth profile & select the environment

```bash
pac auth create \
  --name CODE-APP-DEV \
  --url https://<env>.crm.dynamics.com \
  --applicationId <client-id> \
  --tenant <tenant-id>

pac auth list
pac env select --environment 00000000-0000-0000-0000-000000000000
```

Use `--device-code` if interactive login fails and make sure you’re signed in with the same Microsoft Entra account that owns the target environment.

### 5. Initialize or refresh Dataverse metadata

```bash
pac code add-data-source -a dataverse -t prefix_table_entity_a
pac code add-data-source -a dataverse -t prefix_table_entity_b
pac code add-data-source -a dataverse -t prefix_table_entity_3
```

PAC updates `src/generated/**`, `.power/schemas/**`, and `power.config.json`. Commit these outputs.

### 6. Run locally inside the host

1. Start the Power Apps host bridge:
   ```bash
   pac code start --path .
   ```
2. In another terminal, run Vite (port must match `power.config.json`):
   ```bash
   npm run dev -- --port 3000
   ```
3. Open the “Local Play” URL from `pac code start` using the same browser profile that’s already signed into your tenant.

### 7. Build & validate

```bash
npm run lint
npm run build
```

Optional: `npm run preview` for a production-bundle smoke test.

### 8. Push to the environment

```bash
pac code push \
  --environment 00000000-0000-0000-0000-000000000000 \
  --solutionName "companyCodeAppSolution" \
  --publish
```

You can also try the new module:

```bash
npx power-apps push -s "companyCodeAppSolution"
```

### 9. Pack/import a managed solution (optional)

```bash
pac solution add-reference --path .
pac solution pack --zipFile ./dist/solution.zip --folder . --process CanvasApps
pac solution import \
  --path ./dist/solution.zip \
  --environment 00000000-0000-0000-0000-000000000000 \
  --activate-plugins
```

### 10. Helpful references

- [Quickstart: Create a code app from scratch](https://learn.microsoft.com/power-apps/developer/code-apps/how-to/create-an-app-from-scratch)
- [Power Platform CLI docs](https://learn.microsoft.com/power-platform/developer/cli/introduction)
- [Power Apps code components overview](https://learn.microsoft.com/power-apps/developer/component-framework/overview)
- [Vite official guide](https://vite.dev/guide/)

Those ten sections mirror Microsoft’s guidance and capture every command I ran before diving into UI polish.

## Making the Shell Match the Vision

With the plumbing ready, I discovered the “blank slate” reality of code apps: nothing looks like the design until you type every pixel into CSS. Today’s focus was rebuilding the ProjectHub-inspired frame inside a Vite/React app.

I managed to create solid frame of the application with the main landing page which substitutes as dashboard and couple of sub pages with table view with basic functionalities like filtering, sorting and creating new items.

## Architecture Diagram

graph TD
    user[FinCon / Billing Analyst] --> shell[Power Apps shell<br/>Local Play iframe]
    shell --> host[pac code start bridge<br/>power.config.json localAppUrl]
    host --> vite[Vite dev server / Vite build output]

    subgraph React Code App (src/)
        main[main.tsx<br/>bootstraps <App/>]
        layout[AppLayout<br/>nav + sections]
        overview[OverviewPage<br/>dataset health tiles]
        dataset[DatasetPage<br/>filters, forms, table]
        columns[datasetColumns.ts<br/>builds columns/forms from .power schemas]
        loader[datasetLoader.ts<br/>fetchDatasetRecords + projectRecordFromRaw]
        services[Generated Dataverse services<br/>src/generated/services/*]
    end

    vite --> main --> layout
    layout --> overview
    layout --> dataset
    dataset --> columns
    dataset --> loader
    loader --> services
    services --> dataverse[(Dataverse tables<br/>acoe_ia_bimain / ... / categorymapping)]

## App Layout 

Overview Page (src/pages/OverviewPage.tsx:1-64)
├─ Executive summary header (timestamp + CTA)
└─ Tile grid
   ├─ Dataset title/description
   ├─ KPIs: Total / Ready / Blocked counts
   └─ Button opens DatasetPage via onOpenDataset()

Dataset Page (src/pages/DatasetPage.tsx:1-200)
├─ Header
│  ├─ Dataset name/description
│  ├─ KPI pills: Total, Ready, Blocked, Variance
│  └─ “New record” button (hidden for readOnly datasets)
├─ Filter panel
│  ├─ Search (customer/service)
│  ├─ Status dropdown (Draft/Ready/Sent/Blocked)
│  ├─ Billing cycle dropdown (computed from records)
│  ├─ Owner dropdown (computed from records)
│  └─ Reset filters action
├─ ResponsiveTable
│  ├─ Schema-driven columns (desktop + mobile cards)
│  ├─ Sorting (fallback columns only)
│  ├─ Inline badges (status, amount)
│  └─ Row actions (Edit/Delete) when dataset is writable
└─ Modals (writable datasets only)
   ├─ RecordForm (create/edit) – fields from getDatasetFormFields()
   └─ DeleteDialog – confirms removal before onDeleteRecord()

Shared Data/Schema Layer
├─ data/datasetLoader.ts:1-245
│  ├─ datasetCatalog metadata (labels, categories, readOnly flag)
│  ├─ fetchDatasetRecords() – calls generated services (top 100 rows)
│  └─ projectRecordFromRaw() – normalizes CRUD payloads
└─ config/datasetColumns.ts:1-255
   ├─ Parses .power/schemas to build column order + labels
   ├─ Derives mobile card fields and form definitions
   └─ Exposes getters consumed by App.tsx

## What’s Next

This post kicks off a short series documenting my path from “Power Apps maker” to “code app tinkerer.” Up next:

- Wiring the datasets to the generated Dataverse services instead of local mocks.
- Documenting PAC CLI deployment scripts so pushes to Dev/Test feel routine.
- Capturing performance learnings once real data hits the responsive table.

If you are on the same journey, start with the tutorial steps above, expect to tussle at the beginning and brace yourself with patiance towards your AI friends in case you are like me and you do not have so strong skill set in pro code development. Write everything down—you’ll want these breadcrumbs when the next hiccup arrives and when the output is good I recommend to commit in case the next edits are not so great.
