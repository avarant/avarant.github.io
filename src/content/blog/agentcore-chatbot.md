---
title: "AgentCore Chatbot: A Self-Hosted AI Chat Widget on AWS"
description: "An overview of AgentCore Chatbot — an embeddable, JWT-authenticated AI chat widget built on AWS Bedrock AgentCore."
publicationDate: 2026-04-26
tags: ["aws", "bedrock", "agentcore", "claude", "terraform", "ai"]
---

I built [AgentCore Chatbot](https://github.com/avarant/agentcore-chatbot), a self-hosted platform for dropping an AI chat widget onto any site. It runs entirely in your AWS account, authenticates end users via your own OIDC provider, and connects the agent to your tools through MCP.

## What it does

- **Embeddable widget** — one `<script>` tag, Shadow DOM isolated, SSE streaming with markdown rendering
- **Per-user auth** — the widget fetches a JWT from your token endpoint; AgentCore validates it against your OIDC discovery URL
- **Conversation memory** — persisted across sessions via AWS Bedrock AgentCore Memory, keyed by the user identity in the JWT
- **Knowledge base (optional)** — drag-and-drop document upload, embedded via Bedrock and stored in S3 Vectors, retrieved by the agent at query time
- **Multi-site dashboard** — manage multiple deployments from one Cognito-protected UI

## Architecture

Three Terraform stacks:

1. **Agent stack** (per site) — AgentCore Runtime (containerized Python agent on ECR), AgentCore Memory, the widget CDN (S3 + CloudFront), and an optional Bedrock Knowledge Base
2. **Dashboard stack** (deployed once) — Hono API on Lambda + Cognito + Next.js UI, manages all sites via a `SITES_CONFIG` env var
3. **Demo stack** — a full working example

The agent itself is a Python container running the [Strands](https://strandsagents.com/) SDK, calling Claude Sonnet 4.6 via Bedrock with MCP tools and an optional `retrieve` tool for the knowledge base.

## Chat flow

```
Widget → customer token endpoint → JWT
       → AgentCore Runtime (validates JWT against OIDC)
       → Agent (Claude tool loop + MCP + KB retrieve)
       → SSE stream back → widget renders markdown
```

The widget talks to AgentCore directly — no Lambda proxy in the hot path. The agent extracts user identity from the JWT server-side (never trusting client-supplied IDs) and uses it as the AgentCore Memory actor ID, so each user gets their own persistent conversation history.

## Why AgentCore

AgentCore Runtime handles the parts that are tedious to build yourself: JWT validation against arbitrary OIDC providers, header passthrough to the container, autoscaling, and managed conversation memory. You bring a Docker image and a system prompt; it handles the rest.

The main gotcha: AgentCore caches Docker images by tag, so deploys need unique tags (`deploy-<timestamp>`) rather than `latest`. The `deploy-agent.sh` script handles this automatically by tagging, updating `terraform.tfvars`, and running `terraform apply`. Avoid the `update-agent-runtime` CLI — it wipes any config you don't pass explicitly.

## Stack

- **Frontend**: Next.js 15, React 19, Tailwind 4 (static export to S3)
- **API**: Hono on Lambda Function URL behind CloudFront
- **Agent**: Python 3.11 container, Strands SDK, Claude Sonnet 4.6
- **Auth**: Cognito (dashboard), customer's OIDC (end-user chat), API key (programmatic)
- **IaC**: Terraform, AWS provider ~>6.0
- **Monorepo**: pnpm workspaces

Code is on [GitHub](https://github.com/avarant/agentcore-chatbot). Deploy is `./scripts/deploy-all.sh` once you've configured your Terraform variables.
