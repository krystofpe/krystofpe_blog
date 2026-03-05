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

Before I could even touch JSX, I had to give my workstation all the toys a code app expects. If you are starting from scratch, these are the steps I wish someone had printed on a single page:

1. **Install .NET 8 SDK**  
   Download the latest LTS installer from [dotnet.microsoft.com](https://dotnet.microsoft.com/). The PAC CLI bootstrappers look for a modern .NET runtime, so verify it with:
   ```sh
   dotnet --info
   ```
2. **Install the Microsoft Power Platform CLI (PAC CLI)**  
   Use `winget install Microsoft.PowerApps.CLI` (or the MSI) and confirm it’s on your path:
   ```sh
   pac --version
   ```
3. **Create an authentication profile**  
   ```sh
   pac auth create --name contoso-dev --url https://make.powerapps.com --applicationId <client-id> --tenant <tenant-id>
   pac auth list
   pac auth select --name contoso-dev
   ```
   This keeps solution exports and `pac paportal` commands scoped to the correct tenant without re‑signing in every time.
4. **Enable code components for the environment**  
   Inside Power Platform admin center, open **Settings → Features → Power Apps component framework for canvas apps** and flip it on. Repeat per environment—Dev/Test/Prod don’t inherit.
5. **Provision the solution container up front**  
   Create (or reuse) a managed ALM-ready solution, e.g. `company.CodeApp.Shell`. From the repo root:
   ```sh
   pac solution init --publisher-name company --publisher-prefix cmp --package-output-path ./dist
   ```
6. **Bootstrap the code app workspace**  
   I used Vite + React:
   ```sh
   npm create vite@latest code-app-shell -- --template react-ts
   cd code-app-shell
   npm install
   ```
   Then install the Power Apps helper packages declared in `package.json` (in my case `@microsoft/power-apps`, `@microsoft/power-apps-vite`).
7. **Build once before touching Power Apps**  
   ```sh
   npm run build
   ```
   The first successful Vite build shakes out TypeScript or path issues that would otherwise surface only after a slow solution import.
8. **Add the bundle to the solution**  
   ```sh
   pac pcf push --publisher-prefix cmp --environment <env-id> --solution-name company.CodeApp.Shell
   ```
   When the push completes, the component appears inside the chosen solution and can be inserted into a canvas app like any other control.

That workflow—install, auth, enable, build, push—is the baseline I’ll reference in the rest of this series.

## Making the Shell Match the Vision

With the plumbing ready, I discovered the “blank slate” reality of code apps: nothing looks like the design until you type every pixel into CSS. Today’s focus was rebuilding the ProjectHub-inspired frame inside a Vite/React app.

## What’s Next

This post kicks off a short series documenting my path from “Power Apps maker” to “code app tinkerer.” Up next:

- Wiring the datasets to the generated Dataverse services instead of local mocks.
- Documenting PAC CLI deployment scripts so pushes to Dev/Test feel routine.
- Capturing performance learnings once real data hits the responsive table.

If you are on the same journey, start with the tutorial steps above, expect to tussle with CSS grids, and write everything down—you’ll want these breadcrumbs when the next hiccup arrives.
