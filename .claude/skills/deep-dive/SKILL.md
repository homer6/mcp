---
name: deep-dive
description: Author a deep-dive companion doc for a topic in this bridge repo, anchored to upstream submodule sources or normative external standards, with Graphviz diagrams. Use when the user asks for a "deep dive" on a topic, says "really understand X", or asks "why is X designed this way" and there's relevant material in `modules/` or in a referenceable RFC/spec. The output is a long-form Markdown file at `docs/topics/<topic>/<topic>-deep-dive.md` paired with `.dot` + `.png` diagrams in `docs/topics/<topic>/diagrams/`, with every load-bearing claim citing a file path, SEP number, or RFC number.
---

# deep-dive

Codifies the workflow that produced `docs/topics/mcp/mcp-deep-dive.md`, `docs/topics/mcp/mcp-draft-spec.md`, `docs/topics/llms-txt/llms-txt-deep-dive.md`, and `docs/topics/well-known/well-known-deep-dive.md`. The pattern is: **read upstream end-to-end → render diagrams of the load-bearing concepts → write the doc anchored to specific upstream paths or normative standards**.

A deep-dive is the *companion* to an introductory explainer (`what-is-<topic>.md`). Don't duplicate the introduction — link to it and go further into the *why*.

## When to invoke

Strong signals from the user:

- "deep dive on X", "really understand X", "complete walkthrough of X"
- "why does X look like this?", "what problem does X solve?"
- "describe the upcoming Y", "compare draft vs Y"
- An existing `docs/what-is-X.md` exists but the user wants more depth

Skip when:

- The user wants only an introduction or surface explainer (write `docs/what-is-X.md` instead).
- The topic isn't represented in `modules/` and would have to be sourced entirely from external material — this skill's load-bearing claim is *citation to upstream files in this repo*. Without a module source, the citation pattern doesn't hold.

## Inputs to gather (briefly, only if not obvious)

1. **Topic** — concrete enough to bound the doc (e.g., "the auth flows", "tasks primitive", "the upcoming draft spec"). One topic per doc.
2. **Source module(s)** — usually `modules/modelcontextprotocol/` for MCP work. Confirm if ambiguous.
3. **Whether an introductory `what-is-<topic>.md` already exists.** If yes, link to it; if no, decide with the user whether to author the intro first or skip straight to the deep-dive.

Don't over-question. Topics like "MCP tools" or "the registry" are clear enough to start.

## Workflow

### Step 1 — Inventory the upstream surface

For the chosen topic, identify the relevant files in the module. Typical buckets to consider:

- **Schema source-of-truth**: `modules/<upstream>/schema/<latest>/schema.ts` (or equivalent). Read this directly into your context — it's the authoritative wire format and usually carries inline doc-comments that are quotable rationale.
- **Spec docs**: `modules/<upstream>/docs/specification/<latest>/...`
- **SEPs / design proposals**: `modules/<upstream>/seps/` — often contain candid problem statements you should quote.
- **Conceptual guides**: `modules/<upstream>/docs/docs/learn/...`, `getting-started/...`
- **Anything in extensions, registry, or examples** if the topic touches those.

### Step 2 — Parallelize heavy reads with Explore agents

If the surface is large (many SEPs, deep spec tree), dispatch `Explore` subagents in parallel rather than reading everything yourself. Don't duplicate their work — read different files yourself while they're running.

Brief them with concrete instructions:
- "Read every file in `modules/<x>/seps/` and produce a per-SEP digest with problem-quote, solution, wire-format impact."
- "Survey `modules/<x>/docs/specification/<latest>/` and report the structure with key passages quoted."

Ask them for **direct quotes** from the source for any rationale-flavored claim — those become your anchors.

### Step 3 — Plan the diagrams

Most deep-dives benefit from 4–8 Graphviz diagrams. Good candidates:

- **Architecture or topology**: who talks to whom and over what
- **State machines**: anything with explicit lifecycle states (e.g., task states, connection lifecycle)
- **Message flows / sequences**: agent loops, request-response patterns
- **Comparison tables / matrices**: render with HTML labels for clean tables
- **Anatomy diagrams**: the "what's in this object" visualization
- **Evolution timelines**: spec versions, what landed when
- **Mode / variant comparisons**: e.g., form vs URL elicitation

One diagram per concept; don't pack two stories into one image. Each diagram should be understandable on its own with only its caption for context.

### Step 4 — Author `.dot` source

Place every diagram at `docs/topics/<topic>/diagrams/<topic>-<name>.dot`. Naming conventions:

- Diagrams live alongside the doc that uses them, in the topic's own `diagrams/` folder.
- Filename keeps the topic prefix even though the folder disambiguates — keeps filenames self-describing when referenced elsewhere (`mcp-task-states.dot`, not `task-states.dot`).
- Names are descriptive: `mcp-task-states`, not `diagram3`.

Every `.dot` file starts with a header comment explaining what the diagram shows and why it matters — future renders depend on understanding intent. Example:

```dot
// Task state machine (SEP-1686, schema/2025-11-25)
// Tasks turn any task-augmentable request into a "call now, fetch later"
// operation. Status flows are advisory; actual transitions depend on what
// the underlying request needs.
digraph mcp_task_states {
  ...
}
```

Style notes from the established diagrams:

- `fontname="Helvetica"` for legibility.
- `node [shape=box, style="filled,rounded"]` as the default; use `shape=ellipse` for actors and `shape=note` for sidebar callouts.
- Group related concepts with `subgraph cluster_<name>` and a `style="rounded,dashed"` border.
- Color-code by role consistently across diagrams: blue family for clients/host (`#dbeafe`/`#1e40af`), green family for servers (`#dcfce7`/`#166534`), amber for utilities (`#fef3c7`/`#b45309`), red for errors (`#fee2e2`/`#991b1b`).
- Use HTML labels (`label=<<table>...</table>>`) when you need a small comparison matrix or a structured object diagram — Graphviz handles this well.
- Keep a small `note` callout for capability gates, security warnings, or "see also" references.

### Step 5 — Render

```bash
cd docs/diagrams
for f in *.dot; do dot -Tpng "$f" -o "${f%.dot}.png"; done
```

Render in one batch after authoring all `.dot` files. Verify at least 1–2 of them visually with `Read` to catch layout problems before writing the doc.

Both `.dot` and `.png` are committed — the source so it can be re-rendered, the PNG so the doc renders without local Graphviz.

### Step 6 — Write the deep-dive doc

Create `docs/<topic>-deep-dive.md` following this structure:

```
# <Topic> Deep Dive: <punchy subtitle>

This is the companion to [`what-is-<topic>.md`](what-is-<topic>.md). The introduction
tells you *what*. This document explains *why* — design rationale, the problems each
piece solves, and the tradeoffs the maintainers chose.

Every claim here is anchored to a file in `modules/<upstream>/`. ...

***

## How to read this document

[Sections organized by layer / topic, not by SEP number.]

***

## 1. <First major section>

![alt text](diagrams/<topic>-<name>.png)

[Prose anchored to upstream paths. Use direct quotes (verbatim, in blockquote) from
schema doc-comments and SEP motivation sections for any non-obvious design rationale.]

[Reference schema interfaces by path:line, e.g. `schema.ts:1117–1131`.]
[Reference SEPs by number, e.g. **SEP-1577**.]

***

## N. References

All paths relative to `modules/<upstream>/`.

- **Schema (source of truth)**: `schema/<latest>/schema.ts`
- ...
```

**Citation rules** (this is the load-bearing trust model):

- Every load-bearing factual claim cites either a file path (with line numbers when useful) or a SEP number.
- Quotes from the spec or SEPs are verbatim and in blockquote — don't paraphrase rationale you can quote.
- The final `## References` section lists every upstream file path cited, relative to `modules/<upstream>/`. This is non-optional.
- If a claim can't be anchored, either find a source or remove it. Don't invent.

**Style** matches the existing deep-dives:

- `***` horizontal rules between top-level sections.
- `##` for top-level, `###` for subsections.
- Tables for comparing modes, versions, primitives, error codes.
- Bold key terms on first introduction.
- Each section that covers a concept names the schema interface(s), method(s), and SEP(s) it relates to.

### Step 7 — Update the index

Add the new doc to `README.md` in two places:

1. The repo layout block (`docs/<topic>-deep-dive.md` line).
2. The "Reading order" section, with a one-line summary.

Add each new diagram to the Diagrams table in `README.md`.

If `CLAUDE.md` doesn't yet mention the topic and the deep-dive establishes new domain facts that future docs should respect, also add a bullet to the "MCP domain facts" section (or the analogous facts section for non-MCP topics).

### Step 8 — Don't commit unless asked

Stop here. Surface a summary of what was created. Do **not** create a git commit unless the user explicitly asks for one.

## Examples in this repo

The canonical references (read these to absorb tone and structure):

- `docs/mcp-deep-dive.md` (393 lines) — the flagship example. Eleven sections, six diagrams, every section tied to schema lines and SEPs.
- `docs/mcp-draft-spec.md` (~270 lines) — a *delta* deep-dive comparing draft vs latest stable; demonstrates the at-a-glance change table and the "what did *not* change" section.
- `docs/llms-txt-deep-dive.md` (137 lines) — a smaller deep-dive for a topic without a submodule. Note it includes an explicit "X is not Y" section to head off conflation with related concepts.

Diagrams to study before authoring new ones: `docs/diagrams/mcp-architecture.dot`, `docs/diagrams/mcp-task-states.dot`, `docs/diagrams/mcp-sampling-with-tools.dot`. They demonstrate clusters, color coding, HTML-label tables, and sidebar notes respectively.

## Common pitfalls

- **Paraphrasing where you should quote.** If the schema doc-comment or SEP says it well, blockquote it verbatim. Paraphrasing dilutes the trust model.
- **Citing a file you didn't actually read.** Don't list a path in References that you didn't open. The bridge repo's whole premise is verifiable.
- **Mermaid diagrams in our docs.** The upstream module's contributor guide prefers Mermaid; ours uses Graphviz. The upstream `CLAUDE.md` auto-loads when reading inside `modules/` — ignore that guideline for our own docs.
- **Restating the introduction.** A deep-dive is *companion* to `what-is-<topic>.md`. Link, don't duplicate.
- **Over-broad topics.** "MCP" is too broad; "the MCP schema" is too broad. "The Tasks primitive" or "the auth flows" or "the upcoming draft spec" are right-sized.
- **Skipping Explore agents on big surfaces.** If a topic spans 30 SEPs and a 2,500-line schema, your context will fill up before you write anything. Parallelize.
- **Missing the References section.** Every deep-dive ends with the upstream paths it cites. Without it, future readers can't audit.
- **Over-engineering diagrams.** Keep them readable. If a diagram needs more than a one-paragraph caption to explain, it's probably two diagrams.

## What NOT to put in a deep-dive

- Implementation tutorials ("here's how to write an MCP server in Python"). Those go elsewhere; the deep-dive is *about the spec*, not the SDKs.
- Quickstarts. Wrong genre.
- Marketing-flavored prose. The deep-dive is technical and rationale-driven.
- Speculation not grounded in upstream. If something isn't in the spec or a SEP, don't write it as if it is.

## Quick reference: file layout produced

```
docs/
  <topic>-deep-dive.md            (new)
  diagrams/
    <topic>-<concept-1>.dot       (new)
    <topic>-<concept-1>.png       (new)
    <topic>-<concept-2>.dot       (new)
    <topic>-<concept-2>.png       (new)
    ...
README.md                          (updated: index + diagrams table)
CLAUDE.md                          (updated only if new domain facts emerged)
```
