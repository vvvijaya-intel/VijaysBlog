---
layout: post
title: "How To: Turn GitHub Copilot Chats into Better Azure DevOps User Stories"
date: 2026-08-10
tags: [github-copilot, azure-devops, ado, user-stories, agile, productivity]
---

## The Problem with Handwritten Work Items

A useful engineering conversation often contains everything a work item needs: the user impact, the observed behavior, the constraints, and the checks that would prove the work is complete.

The problem is that this information is easy to lose when someone later creates an Azure DevOps User Story by hand. Titles become vague, acceptance criteria get mixed into the description, parent links are missed, and the original evidence disappears into chat history.

GitHub Copilot Chat can help close that gap by turning the conversation into a structured Azure DevOps work item.

---

## Prerequisite: Azure DevOps MCP Server

Before using this workflow, install and configure the [Azure DevOps MCP Server](https://github.com/mcp/microsoft/azure-devops-mcp) in GitHub Copilot. The MCP server provides the Azure DevOps tools that Copilot uses to retrieve parent work items, create child stories, update fields, and read the completed story back for verification.

For this walkthrough, I used the local MCP server installation steps.

Without the server available, Copilot can still help draft the story text, but it cannot perform the Azure DevOps work-item operations from the chat.

---

## The Workflow

With an ADO story-creation skill and Azure DevOps tools available, the flow can be conversational:

1. Describe the problem in GitHub Copilot Chat.
2. Ask Copilot to create the User Story.
3. Copilot retrieves the parent Feature and confirms its metadata.
4. Copilot creates the story as a child of that Feature.
5. Copilot adds the acceptance criteria to the dedicated ADO field.
6. Copilot assigns the story, applies the requested story points or tags, and aligns the area and iteration paths.
7. Copilot reads the story back and verifies the result.

The important part is that the conversation is not just copied into a description field. It is translated into the fields and relationships that make the work item useful to the team.

---

## A Simple Prompt

A prompt can be as direct as:

> Create an Azure DevOps User Story for this issue: the Databook card uses a shared environment token instead of the signed-in user's Azure SSO token. Use the default parent and assign it to me. Set it to Active with 1 story point.

The skill turns that request into a story with a concise title, a user-story statement, context about the observed issue, and observable acceptance criteria.

A follow-up can also provide more context:

> Add acceptance criteria confirming that the product catalog, product search, filter search, and query endpoints all use the browser's Azure SSO token and reject requests without a bearer token.

This keeps the technical detail connected to the outcome the team needs to verify.

---

## Better Descriptions

A strong description separates the purpose of the work from the evidence behind it.

For example:

```markdown
## User story
As a Sentry user, I want Databook requests to use my Azure SSO token, so that semantic-model access is authenticated as me.

## Observed issue
The direct Databook card and normal chat path did not use the same authentication flow. The card could rely on a server-side token path, which made interactive access inconsistent.

## Notes
The browser already acquires a delegated token through the existing sign-in flow. The work should cover catalog loading, product and filter searches, and query execution.
```

This is more useful than a description such as “Fix Databook authentication” because it records who needs the capability, what was observed, and what belongs in scope.

---

## Acceptance Criteria as a Contract

Acceptance criteria belong in the dedicated `Microsoft.VSTS.Common.AcceptanceCriteria` field rather than being appended to the description.

For the authentication example, the criteria might be:

- The Databook card sends the browser token with its initial catalog request.
- Product search and additional filter-value requests send the same per-request token.
- Databook query execution uses the browser token for the Fabric call.
- Databook and KESTREL endpoints reject requests without a bearer token.
- No interactive runtime path falls back to a shared environment token.
- Focused automated coverage verifies token forwarding, missing-token rejection, cleanup, and existing response contracts.

These are observable outcomes. They give engineering a clear implementation target and give reviewers a practical checklist for deciding whether the story is complete.

---

## Why This Helps the Team

Using GHCP Chat and Azure DevOps together can improve more than the wording of one story:

- **Less context loss:** the issue, evidence, and constraints travel together.
- **More consistent stories:** teams use a repeatable structure instead of starting from a blank form.
- **Stronger acceptance criteria:** criteria describe behavior that can be checked, tested, or demonstrated.
- **Better hierarchy:** the story is created under the intended Feature instead of becoming an orphan.
- **Cleaner ownership:** assignment, area, iteration, tags, and story points can be set during creation.
- **Faster refinement:** the team can review a concrete story while the original conversation is still fresh.
- **Better auditability:** the story can record what was verified after creation.
- **Fewer duplicate work items:** read-back verification catches a failed update before someone retries by creating another story.

The result is a better handoff between discovery, planning, implementation, and review.

---

## Keep the Human in the Loop

Copilot can structure the work, but the team still owns the decision. Review the title, scope, owner, acceptance criteria, and metadata before implementation begins.

A useful pattern is to ask Copilot to create the story, read it back, and show the resulting ID and link. The team can then refine the story in Azure DevOps using the same shared understanding from the conversation.

The goal is not to automate judgment. It is to make the judgment already present in the conversation easier to preserve, review, and act on.

---

## Summary

GitHub Copilot Chat can be a practical bridge between technical conversations and Azure DevOps planning. With a focused skill and the right ADO tools, a chat can become a User Story with:

- A clear user outcome
- Concrete observed context
- Dedicated acceptance criteria
- Correct parent hierarchy
- Consistent ownership and iteration metadata
- Story points and useful tags
- A read-back verification step

That gives the team a repeatable way to turn good conversations into work items that are ready for refinement.
