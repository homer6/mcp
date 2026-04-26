# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a knowledge / documentation repository about the **Model Context Protocol (MCP)**. It is not a software project — there is no build, lint, or test pipeline. The work here is:

- Exploring the MCP protocol in depth
- Writing long-form explainers and examples in Markdown under `docs/`
- Authoring Graphviz `.dot` source files for diagrams, rendering them to PNG, and embedding the rendered images in the corresponding Markdown

When asked to "document X" or "explain Y" about MCP, the deliverable is almost always a new or updated Markdown file in `docs/`, optionally accompanied by one or more diagrams.

## Diagram workflow (Graphviz)

Diagrams are authored as `.dot` source and rendered to PNG, then referenced from Markdown. Always commit both the `.dot` source and the rendered `.png` so the diagram can be re-rendered later without guessing.

Convention to follow when adding diagrams:

- Place `.dot` source alongside the Markdown that uses it, e.g. `docs/diagrams/<name>.dot` with the rendered output at `docs/diagrams/<name>.png`.
- Render with: `dot -Tpng docs/diagrams/<name>.dot -o docs/diagrams/<name>.png`
- Embed in the Markdown using a relative path: `![alt text](diagrams/<name>.png)`
- Graphviz is installed at `/opt/homebrew/bin/dot` (verified `dot -V` → graphviz 14.0.0).

If a directory like `docs/diagrams/` does not yet exist when you add the first diagram, create it.

## Writing conventions for docs

Existing docs (`docs/what-is-mcp.md`) establish the house style — match it when adding new pages:

- Long-form, explainer tone with `***` horizontal rules separating major sections.
- `##` for top-level sections, `###` for subsections.
- Inline footnote-style citations like `[1][2]` after factual claims sourced from external material. Preserve these when editing; do not strip citation markers.
- Tables for version/spec comparisons.
- Bold key terms on first introduction.

## MCP domain facts to keep consistent across docs

The existing explainer establishes facts that newer docs should not contradict (update the explainer instead if you have better information):

- MCP was introduced by Anthropic in **November 2024** and donated to the **Agentic AI Foundation (AAIF)** under the **Linux Foundation** in **December 2025**.
- Spec versioning is **date-stamped**. Current versions referenced: `2024-11-05` (legacy), `2025-06-18` (stable), `2025-11-25` (latest).
- Architecture is **client-host-server** over **JSON-RPC**, with **STDIO** and **HTTP+SSE** transports.
- The **six core capabilities** are: Tools, Resources, Prompts, Sampling, Roots, Elicitation. New docs that cover capabilities should use these names exactly.
- The `2025-11-25` spec is the cutoff for "modern" features: async Tasks primitive, Sampling-with-Tools, URL-mode Elicitation, M2M OAuth (`client_credentials`).

When today's date matters to a doc (e.g., "as of …"), use the actual current date rather than copying the date from another file.
