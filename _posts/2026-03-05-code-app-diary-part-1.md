---
layout: single
title: "Code App Diary · Part 1: From Blank Vite Shell to hopefully MVP application"
date: 2026-03-05 00:00:00 -0500
categories: [Power Platform]
tags: [Power Apps, Power Apps code components, PAC CLI, Vite, React]
toc: true
excerpt: "My first attempts at building a Power Apps code app from scratch: setting up the toolchain, fighting through prompting via Codex and GitHub Copilot"
---

## Kickstarting a Code App Environment (Tutorial)

Before I could even touch JSX, I had to give my workstation all the toys a code app expects. If you are starting from scratch, these are the steps I wish someone had printed on a single page. I rewrote this section to match Microsoft’s official quickstart plus the exact steps I followed for the Compayn project. It walks through the journey from clean laptop to first deployed build.

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

If you are not using this repo, scaffold Microsoft’s Vite starter:

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
pac code add-data-source -a dataverse -t acoe_ia_bimain
pac code add-data-source -a dataverse -t acoe_ia_bibillingprice
pac code add-data-source -a dataverse -t acoe_ia_bimanual
pac code add-data-source -a dataverse -t acoe_ia_bibilling
pac code add-data-source -a dataverse -t acoe_ia_bibillingagg
pac code add-data-source -a dataverse -t acoe_ia_categorymapping
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
  --solutionName "CompaynCodeAppSolution" \
  --publish
```

You can also try the new module:

```bash
npx power-apps push -s "CompaynCodeAppSolution"
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

## What’s Next

This post kicks off a short series documenting my path from “Power Apps maker” to “code app tinkerer.” Up next:

- Wiring the datasets to the generated Dataverse services instead of local mocks.
- Documenting PAC CLI deployment scripts so pushes to Dev/Test feel routine.
- Capturing performance learnings once real data hits the responsive table.

If you are on the same journey, start with the tutorial steps above, expect to tussle with CSS grids, and write everything down—you’ll want these breadcrumbs when the next hiccup arrives.
