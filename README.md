# mcp

A **bridge repo** for exploring the **Model Context Protocol (MCP)** in depth and the adjacent `llms.txt` proposal.

A bridge repo combines original documentation with upstream source-of-truth, pinned as git submodules under `modules/`, so every claim in our docs can be traced back to a specific file in the upstream spec rather than paraphrased from secondary sources.

## What's here

```
docs/
  what-is-mcp.md          - introductory explainer for MCP
  mcp-deep-dive.md        - rationale for each part of the schema, SEP-by-SEP
  what-is-llms-txt.md     - introductory explainer for llms.txt
  llms-txt-deep-dive.md   - design rationale and efficacy critique
  diagrams/               - Graphviz .dot sources + rendered .png
modules/
  modelcontextprotocol/   - submodule: github.com/modelcontextprotocol/modelcontextprotocol
                            (canonical spec, SEPs, schema TypeScript + JSON)
CLAUDE.md                 - guidance for Claude Code working in this repo
```

## Reading order

If you're new to MCP, read in this order:

1. [`docs/what-is-mcp.md`](docs/what-is-mcp.md) — what MCP is, who uses it, and why it exists.
2. [`docs/mcp-deep-dive.md`](docs/mcp-deep-dive.md) — the schema explained section by section, with diagrams, anchored to the upstream spec and the Spec Enhancement Proposals (SEPs) that introduced each feature.

For `llms.txt`:

1. [`docs/what-is-llms-txt.md`](docs/what-is-llms-txt.md)
2. [`docs/llms-txt-deep-dive.md`](docs/llms-txt-deep-dive.md)

## Diagrams

All diagrams are Graphviz `.dot` source rendered to PNG. Both files are committed so anyone can re-render. Current set:

| Diagram | Topic |
|---|---|
| `mcp-architecture` | Host / Client / Server roles, 1:1 client-server connections |
| `mcp-message-types` | JSON-RPC 2.0 — Request, Response (Result \| Error), Notification |
| `mcp-primitives` | Six primitives plus utilities, grouped by initiator and control model |
| `mcp-lifecycle` | Initialize → Operate → Shutdown |
| `mcp-task-states` | Task state machine (SEP-1686) |
| `mcp-sampling-with-tools` | Server-driven agent loop (SEP-1577) |
| `mcp-elicitation-modes` | Form vs URL elicitation (SEP-1036) plus the `-32042` redirect error |
| `llms-txt-anatomy` | The proposed file's required and optional parts |
| `llms-txt-ecosystem` | Comparison with `robots.txt` / `sitemap.xml` and companion files |

Render the whole set:

```sh
cd docs/diagrams
for f in *.dot; do dot -Tpng "$f" -o "${f%.dot}.png"; done
```

## Working with the submodule(s)

After a fresh clone, populate the modules:

```sh
git submodule update --init --recursive
```

To advance a module to a newer upstream commit:

```sh
git -C modules/modelcontextprotocol fetch
git -C modules/modelcontextprotocol checkout <ref>
git add modules/modelcontextprotocol
git commit -m "bump modelcontextprotocol submodule"
```

Files inside `modules/` are **read-only upstream content**. Don't edit them here; if something needs changing, fix it upstream and bump the submodule pointer.

## License

The bridge repo's original docs and diagrams are released under the same MIT license used by the upstream MCP project. Each submodule retains its own license — see the `LICENSE` file inside each `modules/<name>/` directory.
