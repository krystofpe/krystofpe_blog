---
layout: single
title: "Code App Diary · Part 1: From Zero to Blank Vite Shell"
date: 2026-03-05 00:00:00 -0500
categories: [Power Platform]
tags: [Power Apps, Power Apps code components, PAC CLI, Vite, React]
toc: true
excerpt: "My first attempts at building a Power Apps code app from scratch: setting up the toolchain, fighting through prompting via Codex and GitHub Copilot"
---

I kept bumping into hot takes claiming “canvas apps are dead, long live code apps,” so I decided to find out what the fuss was about. If code apps truly unlock pro-dev agility without leaving the Power Platform behind, I want firsthand proof. This diary is my running log of that experiment.

## Kickstarting a Code App Environment

Before I will touch any JSX, I have to give my workstation all the toys a code app expects. If you are starting from scratch, these are the steps I wish someone had printed on a single page. I rewrote this section to match Microsoft’s official quickstart guid and something extra. It walks through the journey from pretty much clean laptop to first deployed build. I had "pac" installed previously, same goes for .NET but I ended up installing it again or in case of .NET updating it to the version 10.

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
npx degit github:microsoft/PowerAppsCodeApps/templates/vite code-app
cd code-app
npm install
pac code init --displayname "code.app"
```

### 3. Clone & install this repo

```bash
git clone https://github.com/<org>/<repo>.git
cd <repo>/code-app
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
pac code add-data-source -a dataverse -t prefix_table_entity_c
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

You can also try the new module which in the end worked form me best:

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

Those ten sections mirror Microsoft’s guidance and capture every command I ran before pushing blank code app container to the environment.

With the tooling now behaving, I’m wrapping this entry as soon as the skeleton project reaches parity with the official quickstart. That keeps the focus on getting from “blank laptop” to “repeatable code app scaffolding” without drifting into design rabbit holes.

## To Be Continued

Part 2 will cover the actual building and editing experience: how I lean on Codex, GitHub Copilot, and PAC CLI together to evolve screens, wire Dataverse data safely, and recover when the AI guesses wrong. If you are curious about pairing AI tools with Power Apps code workflows, keep an eye on the feed—the next installment is entirely about that collaboration.
