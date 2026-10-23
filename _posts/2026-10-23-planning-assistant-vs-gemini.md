---
layout: post
title: "Gemini Enterprise vs Supply Planning Agent"
date: 2026-10-23
tags: [gemini, supply-planning, planning-assistant, agentic-ai, mcp]
---

# Gemini Enterprise vs Supply Planning Agent

Date: Oct 23, 2026

**Objective:** People may be wondering why use the Supply Planning Agent (sentry.intel.com) at all when I have Google's Gemini Enterprise available now within Intel. So I am writing to clarify some differences between the two and how I expect it to evolve over the coming quarters/years. 

**TLDR: Sentry vs Gemini:** Sentry is purpose-built for supply planning — lower hallucination risk, connects to Intel's data (Kestrel, SPARC R2/R3 semantic models), and delivers consistent team-wide results. Gemini is general purpose. Use both — they complement each other. Sentry is built on MCP, keeping its skills and data portable as AI tools evolve.

| Feature                                                                              | Sentry                                                      | Gemini Enterprise                                      |
| ------------------------------------------------------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------ |
| Use Case                                                                             | Supply Planning Specific                                    | General Purpose                                        |
| Generation                                                                           | 2nd Generation -  Agentic AI                                | 1st Generation - Chat AI                               |
| Generate Images                                                                      | No (intentional)                                            | Yes                                                    |
| Spin up own agent                                                                    | No                                                          | Yes                                                    |
| Hallucination Potential                                                              | Low - tuned for Supply Planning, safer default assumptions. | High - generic use model, AI fills in missing details. |
| Repeatable response for same user action?                                            | Yes                                                         | No                                                     |
| Can connect to Semantic Models including Kestrel?                                    | Yes                                                         | No                                                     |
| Answer plain english supply planning questions based on data inside Semantic Models? | Yes                                                         | No                                                     |
| Improve org. memory                                                                  | In Roadmap, approval pending.                               | No, disparate files/process.                           |
| Support Cycle on Cycle Comparison using Kestrel data?                                | Yes                                                         | No                                                     |
| Agent autonomously selects right action based on ask?                                | Yes, via MCP                                                | No                                                     |
| Chaining - string together multiple actions?                                         | Yes                                                         | Yes                                                    |
| Read-to-Use prepackaged Skills?                                                      | Yes                                                         | Yes                                                    |


Where Planning Assistant agent helps the most? 
1. Individual skills, agents tend to be non-deterministic (different results for different users), brittle for team use (people leave for new position) and isolated. Planning agent aims to reduce these weaknesses. 
2. Skills inside Supply Planning agent are universal AND also customizable enough for various use-cases. Any changes to skills are governed and this provides reliable featureset for the entire team/organization.  
3. As more data and related actions are stored in SPARC platform, single point observations can be cross referenced, stacked together to get more accurate information about trends within the supply picture. 

Future Direction?

In a few years, I expect AI efforts to consolidate inside a single provider. The SPARC team is well positioned to adapt to this as we deliberately chose to use Model-Context-Protocol (MCP) as our backbone. MCP is widely used standard with usage proliferating in the year 2026. When that time eventually arrives, users no longer need to navigate to sentry.intel.com but can continue to **utilize the same skills, workflows, stored historical information from the past**. 