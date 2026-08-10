---
layout: post
title: "How To: Use Databook in Sentry"
date: 2026-08-10
tags: [databook, fabric, supply-chain, planning-agent, github-copilot, ai]
---

## Open Databook Directly

Sentry includes a conversational interface for querying the Databook/KESTREL semantic model. The easiest way to begin is with the Databook card.

Click the chat input and type:

```text
/databook
```

Press **Enter**. Wait a few seconds while Sentry loads the available semantic-model properties into the chat interface. The Databook card then displays its current controls and choices, including Version Week, product type, product or marketing code name, fiscal quarters, Supply/Demand selections, and measures.

![Databook card with live model controls]({{ '/assets/images/databook-card.png' | relative_url }})

You can also provide initial context after the command:

```text
/databook Granite Rapids D HCC BGA XP
```

The initial text helps populate the Databook context before you choose the remaining values and run the query.

---

## Find a Product

In the `SNOPDEMANDPRODUCTNM` field, type part of a product name and select **Find matching products**. Sentry displays a visible list of matching product names after the search completes. Select a product from the list; the list closes and the selected name is placed in the product field.

This lookup does not run the Databook analysis. Choose the remaining controls, then select **Run analysis** when the query is ready.

---

## Ask Databook in Chat

You can also ask Sentry directly in chat. For Databook-related questions, start the prompt with **“In Databook”**. This makes the intended data source explicit while the Databook experience is in pilot mode.

### Compare Version Weeks

> In Databook, compare Version Week 202621 vs 202624 for the Processor product Granite Rapids D HCC BGA XP. Use MpsCmp Stitched Sellable (Ku), MpsCmp Total (Ku), TD Pub (Ku), and CD Pub (Ku). Compare the first six fiscal quarters and show Supply 1, Demand 1, Boh Total, Boh Sellable, their deltas, and the inventory-effect evidence.

> In Databook, compare Version Week 202621 versus Version Week 202624 for the marketing code name Granite Rapids, across the first six fiscal quarters, using MpsCmp Stitched Sellable (Ku), MpsCmp Total (Ku), TD Pub (Ku), and CD Pub (Ku).

### Compare Supply and Demand Selections

Within a Version Week, you can compare multiple supply selections against demand selections.

> In Databook, for week 202624 get the Processor product Granite Rapids D HCC BGA XP. Use MpsCmp Stitched Sellable (Ku), MpsCmp Total (Ku), TD Pub (Ku), and CD Pub (Ku).  show Supply 1, Demand 1, Boh Total, Boh Sellable, their deltas, and the inventory-effect evidence.

---

## Ask Follow-Up Questions

After a Databook result is returned, ask follow-up questions in the same chat. The agent retains the comparison context and can answer questions about a different quarter, product, measure, comparison, chart, or explanation.

---

## Be a Good Data Citizen

Databook can return complete result sets. Avoid asking for every product line, product, quarter, Version Week, and measure in one request; that can produce hundreds of thousands or even 1,000,000 rows, create unnecessary query load, and make the result difficult to review.

> Analyze every single Intel product line across all available Version Weeks, quarters, products, and measures in one result.


Start with a bounded product, Version Week, quarter range, or measure group, then expand in focused steps as needed.
