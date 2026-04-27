# mcp

A **bridge repo** for exploring the **Model Context Protocol (MCP)** in depth, the adjacent `llms.txt` proposal, and the underlying `.well-known/` web standard.

A bridge repo combines original documentation with upstream source-of-truth, pinned as git submodules under `modules/`, so every claim in our docs can be traced back to a specific file in the upstream spec rather than paraphrased from secondary sources.

## What's here

```
docs/
  topics/
    mcp/
      what-is-mcp.md         - introductory explainer for MCP
      mcp-deep-dive.md       - rationale for each part of the schema, SEP-by-SEP
      mcp-draft-spec.md      - complete walkthrough of the upcoming DRAFT-2026-v1 spec
      diagrams/              - Graphviz .dot sources + rendered .png
    llms-txt/
      what-is-llms-txt.md    - introductory explainer for llms.txt
      llms-txt-deep-dive.md  - design rationale and efficacy critique
      diagrams/
    well-known/
      what-is-well-known.md  - introductory explainer for the /.well-known/ standard
      well-known-deep-dive.md - rationale, design tradeoffs, MCP / llms.txt cross-references
      diagrams/
modules/
  modelcontextprotocol/      - submodule: github.com/modelcontextprotocol/modelcontextprotocol
                               (canonical spec, SEPs, schema TypeScript + JSON)
CLAUDE.md                    - guidance for Claude Code working in this repo
.claude/skills/              - project-scoped slash skills
  deep-dive/SKILL.md         - the workflow that produces the deep-dive docs in this repo
```

Each topic gets its own subdirectory under `docs/topics/`. The directory layout is the source of truth for grouping; new topics get added as new sibling folders.

## Reading order

**MCP** — start at the introduction and work down:

1. [`docs/topics/mcp/what-is-mcp.md`](docs/topics/mcp/what-is-mcp.md) — what MCP is, who uses it, and why it exists.
2. [`docs/topics/mcp/mcp-deep-dive.md`](docs/topics/mcp/mcp-deep-dive.md) — the schema explained section by section, with diagrams, anchored to the upstream spec and the Spec Enhancement Proposals (SEPs) that introduced each feature.
3. [`docs/topics/mcp/mcp-draft-spec.md`](docs/topics/mcp/mcp-draft-spec.md) — what's changing in the upcoming `DRAFT-2026-v1` spec relative to `2025-11-25` (mostly tightening: required HTTP headers, MUST-not-standalone server requests, formalized `extensions` slot).

**llms.txt**:

1. [`docs/topics/llms-txt/what-is-llms-txt.md`](docs/topics/llms-txt/what-is-llms-txt.md)
2. [`docs/topics/llms-txt/llms-txt-deep-dive.md`](docs/topics/llms-txt/llms-txt-deep-dive.md)

**.well-known**:

1. [`docs/topics/well-known/what-is-well-known.md`](docs/topics/well-known/what-is-well-known.md)
2. [`docs/topics/well-known/well-known-deep-dive.md`](docs/topics/well-known/well-known-deep-dive.md)

## Diagrams

All diagrams are Graphviz `.dot` source rendered to PNG, stored alongside the doc that uses them under `docs/topics/<name>/diagrams/`. Both `.dot` and `.png` are committed so anyone can re-render. Current set:

### MCP — `docs/topics/mcp/diagrams/`

| Diagram | Topic |
|---|---|
| `mcp-architecture` | Host / Client / Server roles, 1:1 client-server connections |
| `mcp-message-types` | JSON-RPC 2.0 — Request, Response (Result \| Error), Notification |
| `mcp-primitives` | Six primitives plus utilities, grouped by initiator and control model |
| `mcp-lifecycle` | Initialize → Operate → Shutdown |
| `mcp-task-states` | Task state machine (SEP-1686) |
| `mcp-sampling-with-tools` | Server-driven agent loop (SEP-1577) |
| `mcp-elicitation-modes` | Form vs URL elicitation (SEP-1036) plus the `-32042` redirect error |
| `mcp-draft-http-request` | Anatomy of a `DRAFT-2026-v1` Streamable HTTP request — required headers, validation, custom `Mcp-Param-{Name}` |

### llms.txt — `docs/topics/llms-txt/diagrams/`

| Diagram | Topic |
|---|---|
| `llms-txt-anatomy` | The proposed file's required and optional parts |
| `llms-txt-ecosystem` | Comparison with `robots.txt` / `sitemap.xml` and companion files |

### .well-known — `docs/topics/well-known/diagrams/`

See [`docs/topics/well-known/well-known-deep-dive.md`](docs/topics/well-known/well-known-deep-dive.md) for the diagram set.

Render every diagram in the repo:

```sh
find docs/topics -name '*.dot' -execdir sh -c 'dot -Tpng "$0" -o "${0%.dot}.png"' {} \;
```

Or render one topic's diagrams:

```sh
cd docs/topics/mcp/diagrams
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
