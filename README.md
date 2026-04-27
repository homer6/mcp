# mcp

A **bridge repo** for exploring the **Model Context Protocol (MCP)** in depth, the adjacent `llms.txt` proposal, the underlying `.well-known/` web standard, **Claude Code Routines** (cloud-scheduled autonomous Claude Code sessions), and **Claude Managed Agents** (the Anthropic Platform's hosted agent harness).

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
    routines/
      what-is-routines.md    - introductory explainer for Claude Code Routines
      routines-deep-dive.md  - design rationale, trigger model, autonomy & safety choices
      diagrams/
    managed-agents/
      what-is-managed-agents.md   - introductory explainer for Claude Managed Agents
      managed-agents-deep-dive.md - 4-resource model, event stream, persistence, comparison with adjacent products
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

**Routines** (Claude Code cloud-scheduled autonomous sessions):

1. [`docs/topics/routines/what-is-routines.md`](docs/topics/routines/what-is-routines.md)
2. [`docs/topics/routines/routines-deep-dive.md`](docs/topics/routines/routines-deep-dive.md) — trigger model, execution autonomy, billing carve-outs, and how routines compare to `/loop`, Desktop scheduled tasks, and GitHub Actions.

**Managed Agents** (Anthropic Platform's hosted agent harness):

1. [`docs/topics/managed-agents/what-is-managed-agents.md`](docs/topics/managed-agents/what-is-managed-agents.md)
2. [`docs/topics/managed-agents/managed-agents-deep-dive.md`](docs/topics/managed-agents/managed-agents-deep-dive.md) — the four-resource model (Agent / Environment / Session / Events), bidirectional event stream, vaults for MCP OAuth, persistence model, and comparison with the Messages API, Routines, and Claude Code on the web.

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

| Diagram | Topic |
|---|---|
| `well-known-anatomy` | URI shape (`/.well-known/<suffix>`) plus IANA registry entry and status tiers (Permanent vs Provisional) |
| `well-known-mcp-auth-discovery` | MCP's auth discovery chain: 401 → `oauth-protected-resource` (RFC 9728) → `oauth-authorization-server` (RFC 8414) → token request |
| `well-known-vs-llms-txt` | Why `llms.txt` is *not* a well-known URI, and what would change if it were |

### Routines — `docs/topics/routines/diagrams/`

| Diagram | Topic |
|---|---|
| `routines-trigger-model` | Three composable trigger types (Schedule / API / GitHub) attached to one saved routine config |
| `routines-execution-flow` | What happens when a trigger fires: cap check → cloud env → repo clone → autonomous session → branch policy → session record |
| `routines-vs-similar` | Comparison matrix: Routines vs Desktop scheduled tasks vs `/loop` vs GitHub Actions |

### Managed Agents — `docs/topics/managed-agents/diagrams/`

| Diagram | Topic |
|---|---|
| `managed-agents-architecture` | Four-resource model (Agent / Environment / Session / Events) plus independent resources (Files, Memory, Vaults) |
| `managed-agents-event-model` | Bidirectional `{domain}.{action}` event taxonomy: user / agent / session / span events |
| `managed-agents-session-states` | Session state machine: `idle` ↔ `running`, `rescheduling`, `terminated`, plus archive/delete |
| `managed-agents-vs-messages` | Comparison matrix: Managed Agents vs Messages API vs Routines vs Claude Code on the web |

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
