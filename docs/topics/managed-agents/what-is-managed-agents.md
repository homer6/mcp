# Claude Managed Agents: What They Are and How They Work

## Overview

**Claude Managed Agents** is a pre-built, configurable agent harness that runs in Anthropic-managed cloud infrastructure. Rather than build your own agent loop, tool execution layer, and runtime sandbox, you create an *agent* once, attach it to an *environment*, and start a *session* — Claude then autonomously reads files, runs commands, browses the web, and executes code in a managed container while your application sends and receives events.[1]

The product sits at a deliberately different abstraction level from the **Messages API**: Messages gives you direct model prompting and asks you to build the loop; Managed Agents gives you the loop, the sandbox, and the observability hooks built-in.[1]

| | Messages API | Claude Managed Agents |
|---|---|---|
| **What it is** | Direct model prompting access | Pre-built, configurable agent harness that runs in managed infrastructure |
| **Best for** | Custom agent loops and fine-grained control | Long-running tasks and asynchronous work |

The harness includes built-in prompt caching, conversation **compaction**, and other performance optimizations for long-running sessions.[1]

The feature is currently in **beta**. All endpoints require the `managed-agents-2026-04-01` beta header (the SDK sets it automatically). Two sub-features — *outcomes* and *multi-agent* — are in research preview behind a separate access request.[1]

***

## Core concepts

The product is built around four resources:[1]

| Concept | Description |
|---------|-------------|
| **Agent** | The model, system prompt, tools, MCP servers, and skills — created once, referenced by ID across sessions, and **versioned** |
| **Environment** | A configured cloud container template (pre-installed packages, network access rules, mounted files) |
| **Session** | A running agent instance within an environment, performing a specific task and generating outputs |
| **Events** | Messages exchanged between your application and the agent (user turns, tool results, status updates) |

A few additional resources are kept independent of session lifecycle:[2]

- **Files** — shared file storage
- **Memory stores** — cross-session memory
- **Vaults** — OAuth credentials for MCP tools (Anthropic refreshes the tokens for you)

These survive when a session is deleted; deletion only removes the session record, its event log, and its container.[2]

***

## How a session works

The lifecycle is simple but slightly different from a single-call API:[1]

1. **Create an agent** — define the model, system prompt, tools, MCP servers, skills.
2. **Create an environment** — configure the cloud container with packages, network rules, mounted files.
3. **Start a session** — `POST /v1/sessions` with `agent` (ID or `{id, version}` to pin) and `environment_id`. This *provisions* the container and agent but **does not start any work**.[2]
4. **Send a `user.message` event** — this kicks off the agent. The session now drives the actual execution.[2]
5. **Stream the response** — the agent runs autonomously, emitting `agent.message`, `agent.tool_use`, `agent.tool_result`, `agent.thinking`, and other events back via SSE.[3]
6. **Steer or interrupt** — send more `user.message` events to guide it, or a `user.interrupt` to stop it mid-execution.[3]

The session *is* a state machine: `idle` (waiting), `running` (processing), `rescheduling` (transient retry), `terminated` (unrecoverable error). Status changes are themselves events you can subscribe to.[2][3]

***

## Built-in tools

Managed Agents ships with a comprehensive built-in tool set:[1]

- **Bash** — run shell commands in the container
- **File operations** — read, write, edit, glob, grep
- **Web search and fetch** — search the web and retrieve content from URLs
- **MCP servers** — connect to external tool providers
- **Custom tools** — your application can expose its own tools and respond via `user.custom_tool_result` events

The full configuration surface lives at `/docs/en/managed-agents/tools`.[1]

***

## Event model in one minute

Communication is event-based, with type strings following a `{domain}.{action}` convention:[3]

- **User events** (you → session): `user.message`, `user.interrupt`, `user.custom_tool_result`, `user.tool_confirmation`, `user.define_outcome` (research preview)
- **Agent events** (session → you): `agent.message`, `agent.thinking`, `agent.tool_use`, `agent.tool_result`, `agent.mcp_tool_use`, `agent.mcp_tool_result`, `agent.custom_tool_use`, `agent.thread_context_compacted` (compaction notification), plus multi-agent thread events
- **Session events**: `session.status_*` (running / idle / rescheduled / terminated), `session.error`, plus outcome and thread events
- **Span events**: `span.model_request_start` / `_end` (with token usage), plus outcome evaluation spans

Every event carries a `processed_at` timestamp; `null` means the harness has queued the event behind preceding ones.[3]

***

## Authentication and identity

- **API-key authenticated.** Standard Anthropic API key in the `x-api-key` header, plus `anthropic-version: 2023-06-01` and the beta header.[2]
- **MCP authentication via vaults.** When the agent uses MCP tools that need OAuth (Slack, Google Drive, etc.), pass `vault_ids` at session creation. The vault holds the credentials; Anthropic manages token refresh automatically.[2]

***

## Use cases the docs call out

The overview frames Managed Agents as best for:[1]

- **Long-running execution** — tasks that run for minutes or hours with multiple tool calls
- **Cloud infrastructure** — secure containers with pre-installed packages and network access
- **Minimal infrastructure** — no need to build your own agent loop, sandbox, or tool execution layer
- **Stateful sessions** — persistent file systems and conversation history across multiple interactions

***

## Rate limits and billing

Managed Agents endpoints are rate-limited per organization:[1]

| Operation | Limit |
|-----------|-------|
| Create endpoints (agents, sessions, environments, etc.) | 300 req/min |
| Read endpoints (retrieve, list, stream, etc.) | 600 req/min |

Standard Anthropic tier-based rate limits and organization-level spend limits also apply on top.[1]

***

## SDK support

Available across the standard Anthropic SDK matrix: Python, TypeScript, C#, Go, Java, PHP, Ruby. All under a `client.beta.sessions` namespace (and corresponding sub-namespaces for events, agents, environments, vaults).[2]

***

## Beta status and access

- **Beta header required**: `anthropic-beta: managed-agents-2026-04-01` on every request.[1]
- **Enabled by default** for all API accounts.[1]
- **Outcomes** and **multi-agent** are gated research-preview features — request access via Anthropic's intake form.[1]

> "Behaviors may be refined between releases to improve outputs."[1]

***

## Branding guidelines

Partners building on Managed Agents have specific branding rules. Allowed labels include "Claude Agent", "Claude" (when within a menu already labeled "Agents"), and `"{YourAgentName} Powered by Claude"`. Names like "Claude Code", "Claude Code Agent", "Claude Cowork", and "Claude Cowork Agent" are not permitted; products must maintain their own branding and not appear to be Claude Code, Claude Cowork, or any other Anthropic product.[1]

***

## Related Anthropic surfaces

- **Messages API** — direct prompting; you build the loop. The right choice when you need fine-grained control.[1]
- **Claude Code Routines** — schedule and trigger Claude Code cloud sessions on cron, API, or GitHub events. See [`../routines/what-is-routines.md`](../routines/what-is-routines.md).
- **MCP** — the protocol Managed Agents uses to talk to external tool providers. See [`../mcp/what-is-mcp.md`](../mcp/what-is-mcp.md).

The deep-dive companion to this doc explains where each fits and what design choices distinguish them.

***

## References

1. [Claude Managed Agents overview](https://platform.claude.com/docs/en/managed-agents/overview.md) — the official Anthropic Platform docs (April 2026).
2. [Start a session](https://platform.claude.com/docs/en/managed-agents/sessions.md) — session lifecycle, creation, archive, delete.
3. [Session event stream](https://platform.claude.com/docs/en/managed-agents/events-and-streaming.md) — full event model, types, send/stream/interrupt patterns.
