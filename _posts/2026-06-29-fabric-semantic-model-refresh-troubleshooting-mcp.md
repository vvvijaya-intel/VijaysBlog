---
layout: post
title: "How To: Utilize MCP for Troubleshooting Fabric Semantic Model Refresh"
date: 2026-06-29
tags: [fabric, power-bi, semantic-model, mcp, snowflake, troubleshooting, github-copilot]
---

## What is MCP?

**Model Context Protocol (MCP)** is an open standard that allows AI assistants to connect to external tools and data sources. In VS Code, GitHub Copilot can use MCP to interact with Azure, Fabric, databases, and more — turning your AI assistant into a powerful troubleshooting partner.

In this post, I'll show how I used GitHub Copilot with MCP to diagnose and fix a failing semantic model refresh in Microsoft Fabric — entirely through natural language conversation.

---

## The Scenario

One of our semantic models in Fabric suddenly stopped refreshing. Instead of clicking through multiple portal screens to investigate, I opened VS Code and had a conversation with GitHub Copilot.

---

## Step 1 — Connect to Fabric

I started by authenticating with Azure (a one-time setup):

```powershell
az login
```

Then I simply asked Copilot:

> **Me:** "List my Fabric workspaces"

Copilot automatically called the Fabric REST API and returned a formatted list of all 21 workspaces I have access to. No need to remember API endpoints or write any code.

> **Me:** "Connect to the Base workspace"

Copilot retrieved all items in the workspace — reports, semantic models, lakehouses, pipelines, notebooks — and presented them in a clean summary.

---

## Step 2 — Investigate the Failing Model

> **Me:** "List all semantic models"

Copilot filtered the workspace items and showed me 40 semantic models with their IDs.

> **Me:** "The Simplified SnOP Semantic Model is having refresh issues. Can you tell me the reason?"

This is where MCP shines. Copilot:
1. Called the Power BI refresh history API
2. Retrieved the last 10 refresh attempts
3. Parsed the error messages
4. Presented a clear diagnosis:

**Root Cause:** Invalid Snowflake credentials

| Time (UTC) | Status |
|------------|--------|
| 19:53 | ❌ Failed |
| 17:31 | ❌ Failed |
| 17:03 | ❌ Failed |
| 16:31 | ✅ Completed |

Copilot identified that refreshes were working until ~16:45 UTC, then started failing — indicating the credentials expired around that time.

---

## Step 3 — Identify the Connection Details

> **Me:** "What account is the connection logging in as?"

Copilot queried the data source configuration and returned:

| Property | Value |
|----------|-------|
| Data Source Name | Snowflake Connection |
| Server | `your-account.privatelink.snowflakecomputing.com` |
| Warehouse | `your_warehouse` |
| Credential Type | Basic (username/password) |

The API doesn't expose the username for security reasons, but Copilot explained how to find it in the Fabric portal.

---

## Step 4 — Verify and Fix Credentials

> **Me:** "The username was YOUR_SERVICE_ACCOUNT. How do I test if the creds are expired?"

Copilot suggested several options:
- Log into Snowflake directly with the service account
- Check Snowflake audit logs for failed login attempts
- Use Power BI Desktop to test the connection

I logged into Snowflake using the service account credentials and confirmed the password had expired. I reset the password through our **internal password management site** following security protocols.

---

## Step 5 — Update Credentials in Fabric

After resetting the Snowflake password, I updated the credentials in Fabric:

1. Go to **Fabric Portal** → Your workspace
2. Click **Settings** (gear icon) → **Manage connections and gateways**
3. Find your Snowflake connection
4. Click **Edit credentials**
5. Enter the new password
6. Click **Save**

---

## Step 6 — Re-enable Scheduled Refresh

> **Me:** "Can you restart scheduled refresh on this semantic model?"

Copilot checked the refresh schedule and found it was **disabled** — Fabric automatically disables scheduled refresh after repeated failures.

Copilot attempted to re-enable it via API but lacked the necessary permissions (403 Forbidden). It then provided clear portal steps:

1. Go to **Fabric Portal** → Your workspace
2. Find your **Semantic Model** → click **⋮** (more options) → **Settings**
3. Expand the **Refresh** section
4. Toggle **Scheduled refresh** to **On**
5. Verify the schedule (days, times, timezone)
6. Click **Apply**

The semantic model now resumes automatic refreshes on the configured schedule.

---

## Why MCP Changes Everything

| Traditional Approach | MCP Approach |
|---------------------|--------------|
| Navigate to Fabric portal | Ask "list my workspaces" |
| Click through settings menus | Ask "why is this model failing?" |
| Search docs for API endpoints | Copilot knows the APIs |
| Write PowerShell/Python scripts | Natural language conversation |
| Parse JSON error responses manually | Copilot summarizes the root cause |

The entire troubleshooting session took **under 5 minutes** — from identifying the problem to understanding exactly what needed to be fixed.

---

## Summary

Using GitHub Copilot with MCP, I was able to:

1. ✅ Connect to my Fabric workspace with a simple request
2. ✅ List and filter semantic models
3. ✅ Retrieve detailed refresh history with error messages
4. ✅ Get a clear diagnosis (expired Snowflake credentials)
5. ✅ Identify the service account and connection details
6. ✅ Reset the password securely via our internal password management site
7. ✅ Update credentials in Fabric
8. ✅ Re-enable automatic scheduled refresh

MCP turns your AI assistant from a code generator into an **operational partner** that can query systems, analyze results, and guide you through fixes.

---

## Useful Commands Reference

For those who want to script these operations, here are the underlying API calls Copilot uses:

| Task | API Endpoint |
|------|--------------|
| List workspaces | `GET https://api.fabric.microsoft.com/v1/workspaces` |
| List workspace items | `GET https://api.fabric.microsoft.com/v1/workspaces/{id}/items` |
| Get refresh history | `GET https://api.powerbi.com/v1.0/myorg/groups/{workspaceId}/datasets/{datasetId}/refreshes` |
| Get data sources | `GET https://api.powerbi.com/v1.0/myorg/groups/{workspaceId}/datasets/{datasetId}/datasources` |
| Get refresh schedule | `GET https://api.powerbi.com/v1.0/myorg/groups/{workspaceId}/datasets/{datasetId}/refreshSchedule` |
| Trigger manual refresh | `POST https://api.powerbi.com/v1.0/myorg/groups/{workspaceId}/datasets/{datasetId}/refreshes` |

**PowerShell example:**

```powershell
# Get access token
$token = az account get-access-token --resource https://api.fabric.microsoft.com --query accessToken -o tsv

# List workspaces
Invoke-RestMethod -Uri "https://api.fabric.microsoft.com/v1/workspaces" `
    -Headers @{Authorization="Bearer $token"} | ConvertTo-Json -Depth 10

# Get refresh history (use Power BI API token)
$pbiToken = az account get-access-token --resource https://analysis.windows.net/powerbi/api --query accessToken -o tsv

Invoke-RestMethod -Uri "https://api.powerbi.com/v1.0/myorg/groups/{workspaceId}/datasets/{datasetId}/refreshes" `
    -Headers @{Authorization="Bearer $pbiToken"} | ConvertTo-Json -Depth 10
```

---

## Pro Tips

- **Start with authentication**: Run `az login` before starting your MCP session
- **Be specific**: "Why is the Simplified SnOP model failing?" works better than "check refreshes"
- **Trust the diagnosis**: MCP parses error JSON and identifies patterns you might miss
- **Use it for exploration**: Ask "what's in this workspace?" to get oriented quickly
- **Combine with portal**: Some actions (like enabling scheduled refresh) still need portal access
