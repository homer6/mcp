# Model Context Protocol (MCP): A Complete Explainer

## Overview

The **Model Context Protocol (MCP)** is an open standard introduced by Anthropic in November 2024 to solve a fundamental problem in AI systems: connecting large language models (LLMs) to the external data, tools, and services they need to be genuinely useful. Think of it as "USB for AI" — just as USB eliminated the need for proprietary hardware connectors, MCP standardizes the plug-and-play interface between LLMs and external systems.[1][2][3]

Within a year of launch, MCP had been adopted by OpenAI, Google DeepMind, Microsoft, Salesforce, and thousands of developers. By December 2025, Anthropic donated MCP to the **Agentic AI Foundation (AAIF)**, a directed fund under the **Linux Foundation**, co-founded by Anthropic, Block, and OpenAI — with support from Google, Microsoft, AWS, Cloudflare, and Bloomberg. This move ensures MCP remains a permanently vendor-neutral, community-governed open standard.[4][5][6]

***

## The Problem MCP Solves: The N×M Integration Nightmare

Before MCP, connecting AI models to external tools was chaotic. If you had *N* tools (Slack, GitHub, databases, APIs) and *M* different AI models (Claude, GPT, Gemini), every combination required a custom adapter — resulting in **N × M bespoke integrations**. This created enormous maintenance burdens and fragmented developer ecosystems.[7][3]

MCP converts this into an **N + M problem**: build one MCP server for a given tool, and any MCP-compatible model can use it without bespoke code. A single GitHub MCP server works with Claude, GPT-4, Gemini, or any other MCP-compatible model — no duplicate connectors required.[1][7]

***

## Current Standard: Specification Versions

MCP uses a **date-stamped versioning scheme**. As of April 2026, the two most relevant versions are:

| Version | Status | Key Additions |
|---------|--------|---------------|
| `2024-11-05` | Legacy | Initial stable release |
| `2025-06-18` | Stable | Structured tool output, OAuth 2.0 Resource Server classification, Resource Indicators (RFC 8707), Elicitation support, Resource links[8][9] |
| `2025-11-25` | Latest/Current | Async Tasks primitive, Enterprise OAuth (M2M, CIMD, XAA), formal Extensions, Sampling with Tools, URL-mode Elicitation, standardized OAuth scopes[10][11] |

The `2025-11-25` release, shipped on the protocol's one-year anniversary, is widely regarded as the "production-ready" milestone — shifting MCP from proof-of-concept to infrastructure that enterprises can depend on. The specification is maintained at `modelcontextprotocol.io` and governed through the Linux Foundation's AAIF.[11][6]

***

## Core Architecture

MCP follows a **client-host-server** model built on **JSON-RPC**, providing a stateful session protocol for context exchange and sampling coordination.[12]

### The Three Roles

- **Host**: The application housing the LLM (e.g., Claude Desktop, Cursor IDE, a custom AI agent). The host manages user authorization, consent, and permission decisions. It coordinates multiple client instances and bridges the LLM's reasoning to the broader integration layer.[13][7]
- **MCP Client**: Embedded inside the host application. Each client maintains a one-to-one protocol-level connection with a single MCP server — handling discovery of tools, resources, and prompts exposed by that server.[14][13]
- **MCP Server**: A standalone service that exposes specific functionality to AI apps. Servers typically focus on one integration point — a GitHub connector, a PostgreSQL interface, a file system reader, etc.[13]

### Transport Layer

MCP supports two primary communication mechanisms:[13]

- **STDIO (Standard Input/Output)**: For local integrations where the client and server run in the same environment.
- **HTTP + SSE (Server-Sent Events)**: For remote connections — HTTP handles client-to-server requests; SSE handles server-to-client responses and streaming. The `2025-11-25` spec also added SSE polling via server-side disconnect for more robust connection management.[15]

***

## The Six Core Capabilities

MCP defines six primitive capabilities that servers can expose to clients:[14]

### 1. Tools
Callable actions such as "read a file," "send a message," or "create a calendar event." Each tool is described by a name, a human-readable description, and a strict input schema. Tools are the primary mechanism for LLMs to *act* on the world — triggering real side effects.[1]

### 2. Resources
Data exposed by the server for the model to ingest — file contents, database rows, API responses. Resources allow the model to extend its effective context beyond its token window by fetching data on demand. Unlike tools, resources are read-only data sources.[1][13]

### 3. Prompts
Reusable, predefined instruction templates that servers supply to clients. They standardize how models perform common tasks, can be versioned centrally, and are explicitly invoked by users or clients — they don't run automatically. Prompts can dynamically reference available tools and resources to build complete workflow scaffolds.[14][13]

### 4. Sampling
A **bidirectional capability** allowing MCP *servers* to request LLM completions from the *client's* model — the flow runs in reverse compared to typical usage. This is critical for agentic workflows: a server-side tool can ask the LLM to generate intermediate reasoning or propose actions before executing them, without the server needing its own API key or model access. The human-in-the-loop design ensures users can review and approve sampling requests before they execute.[16][17][18][14]

The `2025-11-25` spec added **Sampling with Tools**: servers can now include tool definitions in sampling requests, enabling server-side agent loops where the LLM can call tools, receive results, and continue a conversation within a single sampling flow.[17][11]

### 5. Roots
A mechanism for clients to inform servers about the "roots" of the filesystem or data hierarchy they have access to — helping servers understand scope and organize data access appropriately.[14]

### 6. Elicitation
A feature allowing servers to request information from users mid-session — for example, asking for a credential, a configuration value, or user confirmation. The `2025-11-25` spec added **URL-Mode Elicitation**, where sensitive flows (OAuth, payments, API keys) are redirected to a browser rather than handled inside the MCP client — ensuring secrets never transit through the protocol layer.[19][15][11][14]

***

## How a Typical MCP Interaction Works

Below is a simplified end-to-end flow when a user asks an AI assistant for information that requires external data:

1. **User query** arrives at the host application (e.g., "What are my pending orders?")
2. **Host/LLM** determines it needs external data and invokes an MCP tool
3. **MCP client** sends a JSON-RPC request to the relevant MCP server (e.g., an order management system server)
4. **MCP server** executes the function — queries the database, calls an API, reads a file — and returns structured results
5. **MCP client** passes the result back into the model's context
6. **LLM generates a response** informed by the live external data[20]

This entire loop typically completes in seconds, making the model appear to "know" current information it couldn't have from training data alone.[13]

***

## Authentication and Security

Early MCP relied on basic authentication approaches, but the protocol has matured significantly in this area:

- **OAuth 2.0 Resource Server classification** (introduced `2025-06-18`): MCP servers are formally classified as OAuth 2.0 Resource Servers. Clients must include a `resource` parameter (per RFC 8707) when requesting tokens, binding access tokens to specific servers.[9]
- **Machine-to-Machine (M2M) OAuth** (introduced `2025-11-25`): Added `client_credentials` grant support for headless agents and backend services that have no human in the loop.[11]
- **Separation of Resource Provider and Authorization Server**: Earlier specs assumed these were colocated; the updated auth spec cleanly separates them, allowing third-party authorization servers (e.g., Auth0, Okta) to be integrated without custom AS development.[21]
- **MCP Server Security Standard (MSSS)**: A companion open, vendor-neutral security standard with 23 controls across 8 domains (filesystem, process execution, network access, authorization, input validation, logging, supply chain, deployment), with a stable v0.1.0 released in January 2026.[22]

***

## Ecosystem and Adoption

By the time of the Linux Foundation donation in December 2025, MCP had reached:[23]

- **97 million+ monthly SDK downloads** (Python and TypeScript)
- **10,000+ active MCP servers**
- **First-class client support** across ChatGPT, Claude, Cursor, Gemini, Microsoft Copilot, Visual Studio Code, and more

SDKs are available in all major programming languages. The Agentic AI Foundation (AAIF) now co-governs MCP alongside Block's **goose** framework and OpenAI's **AGENTS.md** spec, creating a shared neutral home for open-source agentic AI standards.[24][5]

***

## Why It Matters for Agentic AI

MCP's design philosophy — minimal core, extensible via a formal Extensions mechanism, human-in-the-loop controls, and composable primitives — makes it the foundational plumbing for the next generation of AI agents. Rather than AI models that only generate text, MCP enables agents that can read databases, trigger workflows, call APIs, collect user input mid-task, spawn sub-agents, and operate autonomously for long-running jobs — all within a consistent, auditable, and secure protocol layer.[10][15][11]

The `2025-11-25` spec's introduction of the **async Tasks primitive** is particularly significant: it enables "call-now, fetch-later" execution patterns, with tasks progressing through `working`, `input_required`, `completed`, `failed`, and `cancelled` states — making multi-hour agentic workflows first-class citizens of the protocol.[11]