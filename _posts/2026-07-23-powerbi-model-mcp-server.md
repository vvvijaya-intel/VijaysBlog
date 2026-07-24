---
layout: post
title: "How To: Explore Fabric Semantic Models with the Power BI Model MCP Server"
date: 2026-07-23
tags: [fabric, power-bi, semantic-model, mcp, dax, github-copilot, vscode]
---

## Overview

**MCP (Model Context Protocol)** is an open standard that lets AI assistants connect to external tools and data sources. Think of it as a plug — like a USB port adapter, you install an MCP server for a specific product (Fabric, Azure, Power BI, a database, etc.), and your AI assistant can suddenly *talk* to that product directly from chat. No switching apps, no copy-pasting URLs, no writing API calls by hand.

In this post I'll show how to install the **Power BI Modeling MCP Server** and use it — through plain English in GitHub Copilot — to connect to a live Fabric semantic model, explore its tables, and list every DAX measure.

---

## What is the Power BI Model MCP Server?

The Power BI Model MCP server exposes a set of tools that let AI assistants talk to tabular semantic models — whether they're running locally in Power BI Desktop, on-premises Analysis Services, or in the Fabric cloud via the XML/A endpoint. Operations include:

- Connect / disconnect to a model
- List and inspect tables, columns, relationships, and measures
- Run DAX queries
- Export TMDL / TMSL definitions
- Deploy a model to Fabric

Think of it as a scriptable version of SQL Server Management Studio's Object Explorer, but accessible through natural language in your editor.

---

## Installation

### Prerequisites

- VS Code with the **GitHub Copilot** extension
- A Fabric workspace with a semantic model (or a running Power BI Desktop instance)

### Step 1 — Install the Extension

Install the **[Power BI Modeling MCP Server](https://marketplace.visualstudio.com/items?itemName=analysis-services.powerbi-modeling-mcp)** extension directly from the VS Code Marketplace (publisher: Microsoft, identifier `analysis-services.powerbi-modeling-mcp`).

The extension handles everything — no Node.js, no manual JSON config, no `npx` wrappers. It registers the MCP server automatically when VS Code starts.

### Step 2 — Restart VS Code and Start the MCP Service

After installing the extension, **restart VS Code** fully (not just reload window).

Once restarted, open the Command Palette (`Ctrl+Shift+P`) and search for:

```
MCP: Start Server
```

Select **Power BI Modeling MCP Server** from the list to start it. You only need to do this once per session — VS Code will remember the state.

### Step 3 — Confirm it's Active in Copilot Chat

Open **GitHub Copilot Chat** and switch to **Agent** mode. Click the tools icon and confirm `powerbi-modeling-mcp` appears in the list of available MCP tools.

![vscode-mcp-tools](https://github.com/microsoft/powerbi-modeling-mcp/raw/main/docs/img/vscode-mcp-tools.png)

You can also just ask Copilot directly:

> **Me:** "Is the Power BI modeling MCP server running?"

Copilot will confirm the connection status and list available tools — a quick sanity check before you start.

---

## Connecting to the Simplified SnOP Semantic Model

With the server running, I asked Copilot to connect to the model I care about — the **Simplified SnOP Semantic Model** in the **base** Fabric workspace.

> **Me:** "Connect to simplified snop semantic model in base workspace (fabric)"

Copilot called `ConnectFabric` and within seconds returned:

```
Connected: Fabric-base-Simplified SnOP Semantic Model
Server: powerbi://api.powerbi.com/v1.0/myorg/base
Catalog: Simplified SnOP Semantic Model
```

No connection strings. No XML/A endpoint URLs to look up. Just a natural language instruction.

---

## Exploring the Model Structure

### Tables

> **Me:** "List the data in the model?"

The model came back with **22 tables**:

| Table | Columns | Measures |
|---|---|---|
| Fact | 23 | 1 |
| DimProduct | 54 | 1 |
| DimCorridor | 5 | — |
| DimCustomer | 8 | — |
| DimKeyFigure | 18 | 1 |
| DimProfitCenter | 12 | — |
| DimTimePeriod | 30 | — |
| **MeasuresTable** | — | **36** |
| DimPlanningQtrCalendar | 4 | 3 |
| DimPlanningMonthCalendar | 3 | 1 |
| DimPlanVersion | 16 | — |
| ProfileSignal | 17 | 1 |
| DimLocation | 6 | — |
| BOM | 8 | — |
| LastRefreshTime | 2 | 1 |
| Environments | 2 | 2 |

Plus supporting helper/parameter tables: `Measure Selection`, `Top N Table`, `Units`, `relative quarter`, `Parameter Product`, `Dim YearMonth Relative`.

The star schema is immediately obvious: a central `Fact` table surrounded by dimension tables (`DimProduct`, `DimTimePeriod`, `DimPlanVersion`, etc.) and a dedicated `MeasuresTable` that holds the bulk of the business logic.

### DAX Measures

> **Me:** "Also any DAX measures?"

The server returned **47 measures**, neatly organized into display folders:

**Consensus Demand**
- `Consensus Demand Volume Published` / `Draft`
- `Consensus Demand Dollars Published` / `Draft`
- `*Consensus Demand Published` (starred = key KPI)
- `*Consensus Demand Volume - Base` / `Bear` / `Bull` (scenario planning)
- `Statistical Demand Volume`
- `Consensus Demand Draft Cost` / `Publish Cost`

**ProdCo Revenue (POR — Plan of Record)**
- `ProdCo Revenue Volume` / `Dollars`
- `ProdCo Net ASP`
- `*ProdCo Revenue Volume (CurrentPOR)`

**ProdCo Revenue (LRP — Long Range Plan)**
- `ProdCo LRP Revenue Volume` / `Dollars`

**ProdCo Actuals**
- `ProdCo Actual Revenue Net Volume` / `Dollars`
- `POR ASP`

**Production Actuals**
- `Production Wafer Out Actual Volume` / `Dollars`

**Sales Billing and Return**
- `Net Billing Volume` / `Dollars`

**Price**
- `Average FG Price` / `Average Wafer Price` / `Average NetASP`
- `*FG Price (Latest)` / `*Wafer Price (Latest)`

**Cost**
- `FG Cost` / `Wafer Cost`
- `*FG Cost (Latest)` / `*Wafer Cost (Latest)`

**Utility / Navigation**
- `QoQ Delta%` / `QoQ Delta Qty`
- `Selected Qtr` / `Selected month`
- `DisplayKF`, `LastRefreshTime`, `Address`, `Environment`

**Pending Validation**
- `BY FG Total Supply`

The naming convention tells a clear story: starred measures (`*`) are the approved KPIs surfaced in reports; unstarred variants are the underlying building blocks or scenario variants used during planning.

---

## Why This Workflow Matters

Traditionally, inspecting a semantic model means opening Power BI Desktop, waiting for the model to load, and navigating a GUI. With the Power BI Model MCP server you can:

1. **Script model exploration** — ask questions about any model without opening any app
2. **Audit measures programmatically** — useful before a model review or governance check
3. **Combine with other MCP servers** — in the same conversation I was already querying DuckDB flat files and asking the agent to correlate them with live Fabric data
4. **Run DAX inline** — the server supports arbitrary DAX queries, so you can validate measures directly from chat

The compound effect of having Fabric MCP + Power BI Model MCP + Azure MCP all active in one Copilot session is significant. Investigations that used to require switching between the Fabric portal, Power BI Desktop, and Azure Monitor now happen in a single chat thread.


