# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a **bridge repo** about the **Model Context Protocol (MCP)**. It is a knowledge / documentation repository, not a software project — there is no build, lint, or test pipeline. The work here is:

- Exploring the MCP protocol in depth
- Writing long-form explainers and examples in Markdown under `docs/`
- Authoring Graphviz `.dot` source files for diagrams, rendering them to PNG, and embedding the rendered images in the corresponding Markdown

When asked to "document X" or "explain Y" about MCP, the deliverable is almost always a new or updated Markdown file in `docs/`, optionally accompanied by one or more diagrams.

## Bridge repo concept

This repo is a **bridge repo**: it bridges multiple upstream repositories into a single working whole, alongside our own original docs. Upstream sources live as **git submodules** under `modules/`, one directory per upstream repo. This lets us:

- Keep upstream source-of-truth (specs, SDKs, reference servers) pinned at known commits and visible in this checkout.
- Cross-reference upstream files from our own `docs/` Markdown without copying or paraphrasing.
- Update each module independently when the upstream advances.

### Working with modules

- After cloning this repo fresh, populate the modules with: `git submodule update --init --recursive`
- Add a new module with: `git submodule add <repo-url> modules/<name>` — pick `<name>` to match the upstream repo name (no nesting under org names unless disambiguation requires it).
- Update a module to its upstream tip: `git -C modules/<name> fetch && git -C modules/<name> checkout <ref>`, then commit the new submodule pointer from the bridge-repo root.
- Treat files inside `modules/` as **read-only upstream content**. Do not edit them directly. If something is wrong, fix it upstream or annotate it from our own `docs/`.

### Current modules

| Module | Upstream | Purpose |
|--------|----------|---------|
| `modules/modelcontextprotocol` | `github.com/modelcontextprotocol/modelcontextprotocol` | Canonical MCP specification and modelcontextprotocol.io site content |

### Upstream module CLAUDE.md auto-loads

When reading files inside `modules/modelcontextprotocol/`, that submodule's own `CLAUDE.md` auto-loads into the conversation. Treat it as the upstream maintainers' guidance for *contributing to their repo* — relevant if you're ever proposing a SEP or PR upstream, but **not** the convention for our bridge-repo docs. Two notable differences:

- **Diagrams**: upstream prefers Mermaid for their internal docs; the bridge repo uses Graphviz. Don't switch tools because the upstream CLAUDE.md says Mermaid.
- **Commit messages**: upstream forbids mentioning model names; that's their commit policy, not ours.

## Diagram workflow (Graphviz)

Diagrams are authored as `.dot` source and rendered to PNG, then referenced from Markdown. Always commit both the `.dot` source and the rendered `.png` so the diagram can be re-rendered later without guessing.

Convention:

- `.dot` source and `.png` output both live in `docs/diagrams/<name>.{dot,png}`.
- Render with: `dot -Tpng docs/diagrams/<name>.dot -o docs/diagrams/<name>.png`
- Embed from a doc with: `![alt text](diagrams/<name>.png)` (relative path; docs live in `docs/`).
- Graphviz is at `/opt/homebrew/bin/dot` (graphviz 14.0.0 verified). To render the whole directory at once: `cd docs/diagrams && for f in *.dot; do dot -Tpng "$f" -o "${f%.dot}.png"; done`.
- Naming: use a topic prefix (`mcp-*`, `llms-txt-*`) so the directory groups by subject.
- Keep `.dot` files commented at the top with a one-paragraph rationale — what the diagram is showing and why. Future renders depend on understanding intent.

## Writing conventions for docs

Two doc styles coexist, and new pages should follow whichever is appropriate:

### Introductory explainer (`what-is-X.md`)

- Long-form prose, `***` horizontal rules separating major sections, `##`/`###` hierarchy.
- Inline footnote-style citations like `[1][2]` for facts sourced from external material. **Preserve these when editing; do not strip citation markers.** This style is used by `docs/what-is-mcp.md` and `docs/what-is-llms-txt.md`.
- Tables for version/spec comparisons.
- Bold key terms on first introduction.

### Deep-dive companion (`X-deep-dive.md`)

- Pairs with an introductory explainer — link to it from the top, don't duplicate the introduction.
- **Anchors every load-bearing claim to a file in `modules/<upstream>/`.** Cite by path, with line numbers for schema or with SEP numbers for proposals. Example: `schema.ts:1117–1131`, `SEP-1577`. This is the central trust model: if a claim isn't anchored to upstream, it shouldn't be there.
- Embeds Graphviz diagrams from `docs/diagrams/`.
- Each section names the schema interface, methods, and SEPs it covers.
- A `## References` section at the end listing all upstream paths cited (relative to `modules/<upstream>/`).
- See `docs/mcp-deep-dive.md` for the canonical example.

The split lets the introductory doc stay readable for first-timers while the deep-dive does the heavy lifting on rationale.

## MCP domain facts to keep consistent across docs

The existing explainer establishes facts that newer docs should not contradict (update the explainer instead if you have better information):

- MCP was introduced by Anthropic in **November 2024** and donated to the **Agentic AI Foundation (AAIF)** under the **Linux Foundation** in **December 2025**.
- Spec versioning is **date-stamped**. Current versions referenced: `2024-11-05` (legacy), `2025-06-18` (stable), `2025-11-25` (latest).
- Architecture is **client-host-server** over **JSON-RPC**, with **STDIO** and **HTTP+SSE** transports.
- The **six core capabilities** are: Tools, Resources, Prompts, Sampling, Roots, Elicitation. New docs that cover capabilities should use these names exactly.
- The `2025-11-25` spec is the cutoff for "modern" features: async Tasks primitive, Sampling-with-Tools, URL-mode Elicitation, M2M OAuth (`client_credentials`).

When today's date matters to a doc (e.g., "as of …"), use the actual current date rather than copying the date from another file.
