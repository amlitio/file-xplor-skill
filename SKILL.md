---
name: xplor
description: >
  Build Xplor — a structured cognition engine that transforms documents, codebases, and
  knowledge systems into traversable, AI-queryable knowledge graphs. Three modes:
  (1) Document Mode — PDFs to entity/relationship graphs with AI extraction.
  (2) Code Mode — repositories to function/class/call-chain graphs via AST parsing.
  (3) Skill Graph Mode — structured markdown knowledge systems with wikilinks, YAML
  frontmatter, and Maps of Content into navigable intelligence graphs for AI agents.
  
  Use this skill whenever the user wants to: build a knowledge graph platform, create a
  document intelligence tool, analyze codebases, build an MCP server for code or knowledge,
  create a skill graph engine, build agent cognition infrastructure, create a CLI for
  indexing repos or knowledge bases, build structured knowledge systems, create traversable
  skill networks with wikilinks, build something like DeepWiki but deeper, build "git for
  structured knowledge", create a "graph OS" for AI agents, or anything involving knowledge
  graphs + AI agents + structured traversal.
  
  Also trigger for: Next.js + Firebase + AI extraction, force-directed graph UIs, AST
  parsing with Tree-sitter, MCP server development, wikilink parsing, YAML frontmatter
  extraction, skill authoring tools, agent traversal telemetry, attention scoring,
  progressive disclosure systems, or multi-domain graph fusion. Even partial matches like
  "skill graph", "knowledge engine", "structured cognition", "agent context", "code
  explorer", "repo intelligence" should trigger this skill.
---

# Xplor — Structured Cognition Engine

> People underestimate the power of structured knowledge. Structured knowledge enables
> entirely new classes of applications.

Xplor transforms **documents**, **codebases**, and **knowledge systems** into traversable,
queryable knowledge graphs — then exposes them to AI agents so they don't just follow
instructions, they **understand domains**.

## Why Xplor Exists

Traditional tools give you descriptions. Xplor gives you **relationships**.

When an AI agent asks "What uses this function?" — it gets call chains, downstream
impacts, execution flows. When it asks "How does this therapy technique connect to
attachment theory?" — it traverses the skill graph and pulls in exactly what the
situation requires.

This is the difference between an agent that follows instructions and an agent that
understands a domain.

## Three Modes

| Mode | Input | Extraction Method | Graph Contains |
|------|-------|------------------|----------------|
| **Document** | PDFs, text files | AI entity extraction (Claude) | People, orgs, locations, dates, concepts + relationships |
| **Code** | Git repos, ZIP, local paths | AST parsing (Tree-sitter) | Functions, classes, imports, call chains + CALLS/IMPORTS/EXTENDS/DEFINES |
| **Skill Graph** | Markdown with wikilinks | Structural parsing (frontmatter + links) | Knowledge nodes, MOCs, claims, techniques + semantic connections |

All three modes feed into the same **Knowledge Graph Core** and render through the same
**Explorer UI**. They can also be **fused** — code graph + company docs + domain skills
merged into a single queryable intelligence layer.

---

## Architecture

```
Xplor Platform
├── Interfaces
│   ├── Web UI (Next.js — upload, explore, chat, skill graph browser)
│   ├── CLI (xplor index, xplor mcp, xplor serve, xplor skill)
│   └── MCP Server (query, context, impact, traverse, search)
│
├── Ingestion Pipelines
│   ├── Document Pipeline (PDF → text → AI entity extraction)
│   ├── Code Pipeline (repo → AST → nodes/edges via Tree-sitter)
│   └── Skill Graph Pipeline (markdown → frontmatter + wikilinks → knowledge graph)
│
├── Knowledge Graph Core
│   ├── In-memory graph (nodes + edges + metadata)
│   ├── Traversal engine (BFS/DFS + attention scoring + depth limits)
│   ├── Search (BM25 full-text + semantic similarity)
│   └── Progressive disclosure (index → descriptions → links → sections → full)
│
├── Agent Intelligence Layer
│   ├── Traversal heuristics (attention scoring, path ranking)
│   ├── Context injection (pull relevant nodes for current task)
│   ├── Telemetry (which nodes loaded, why, what was skipped)
│   └── LLM augmentation (Claude API for enrichment + chat)
│
├── Growth Layer
│   ├── Share system (public links, social distribution)
│   ├── Trending (view tracking, viral engine)
│   └── Stripe monetization (Pro tier)
│
└── Deployment
    ├── Vercel (web app)
    └── npm package (CLI + MCP server)
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14 (App Router) |
| Auth | Firebase Auth (Google + Email) |
| Database | Cloud Firestore + in-memory graph |
| AI | Anthropic Claude API |
| PDF Parsing | pdfjs-dist (client-side) |
| Code Parsing | Tree-sitter (WASM for web, native for CLI) |
| Markdown Parsing | gray-matter (frontmatter) + custom wikilink parser |
| MCP Server | @modelcontextprotocol/sdk |
| CLI | commander.js |
| Payments | Stripe |
| Hosting | Vercel + npm registry |

---

## Build Phases

Build in this order. Each phase is independently deployable.

### Phase 1: Document Mode (Web App)
Upload PDFs → AI extracts entities → interactive knowledge graph.
Read `references/document-mode.md` for full implementation.

### Phase 2: Code Mode (Ingestion + CLI)
Index repos via AST parsing, build call-chain graphs.
Read `references/code-mode.md` for the ingestion pipeline.
Read `references/cli-spec.md` for CLI commands.

### Phase 3: MCP Server
Expose the graph to AI agents via Model Context Protocol.
Read `references/mcp-server-spec.md` for tools and integration.

### Phase 4: Skill Graph Mode
Transform structured markdown into navigable intelligence graphs.
Read `references/skill-graph-spec.md` for the complete specification.

### Phase 5: Agent Intelligence Layer
Traversal heuristics, telemetry, progressive disclosure, multi-domain fusion.
Read `references/agent-intelligence.md` for the full spec.

---

## Design System

Read `references/design-system.md` for colors, typography, and component patterns.

### Extended Entity Color Map
```javascript
const TYPE_COLORS = {
  // Document entities
  person: "#FF6B6B", organization: "#4ECDC4", location: "#45B7D1",
  date: "#F7DC6F", money: "#BB8FCE", concept: "#82E0AA",
  document: "#F0B27A", event: "#AED6F1",
  // Code entities
  function: "#61AFEF", class: "#C678DD", variable: "#E5C07B",
  import: "#56B6C2", module: "#98C379", file: "#ABB2BF",
  // Skill graph entities
  skill: "#FF9F43", moc: "#EE5A24", claim: "#A3CB38",
  technique: "#FDA7DF", framework: "#9AECDB", exploration: "#7158e2",
  // Fallback
  default: "#636e72",
};
```

---

## Reference Files

### P0 — Core Contracts (read first when building any component)

| File | Contents |
|------|----------|
| `references/graph-core.md` | **Canonical data model** — unified GraphNode/GraphEdge schemas, ID namespacing, type registry, persistence format |
| `references/progressive-disclosure.md` | **Enforcement rules** — 5 levels, forbidden fields per level, caching, token budgets |
| `references/skill-graph-api.md` | **Skill Graph web API** — ingest, scan, traverse, context, validate, stats endpoints |
| `references/explorer-architecture.md` | **Mode-aware Explorer** — node/edge rendering per kind, sidebar tabs, MOC index, telemetry heatmap |

### P1 — Mode Implementations

| File | Contents |
|------|----------|
| `references/document-mode.md` | PDF pipeline, AI entity extraction prompt, Firestore schema |
| `references/code-mode.md` | AST parsing, Tree-sitter, code entity/relationship extraction |
| `references/skill-graph-spec.md` | Wikilink parsing (sentence-level context), MOCs, traversal, authoring toolkit |
| `references/agent-intelligence.md` | Attention scoring, telemetry, context injection, multi-domain fusion |

### P2 — Infrastructure & Quality

| File | Contents |
|------|----------|
| `references/cli-spec.md` | CLI commands (`xplor index/mcp/serve/skill/fuse`), project structure, package.json |
| `references/mcp-server-spec.md` | 8 MCP tools (query, context, impact, callers, callees, search, traverse, scan) |
| `references/search.md` | Hybrid search — BM25 + cosine similarity + RRF, field indexing weights, fallbacks |
| `references/skill-graph-quality.md` | Quality scoring rubric (-10 broken link, -5 missing desc), cluster detection, MOC coverage |
| `references/security.md` | Upload limits, rate limits, sandboxing, Pro feature gating tiers |
| `references/design-system.md` | Colors, typography, component patterns |
| `references/api-contracts.md` | Document Mode API routes (extract, share, chat, trending, Stripe) |

---

## Positioning

Xplor is:

- **A Knowledge Graph Engine** for code, documents, and structured knowledge
- **A Skill Graph Engine** that turns interconnected markdown into agent-navigable intelligence
- **A Traversal Engine** for structured agent cognition
- **"Git for Structured Knowledge"** — version, traverse, query, and fuse knowledge domains
- **The Graph OS for AI agents** — infrastructure for domain-aware reasoning

Code is one domain. Structured knowledge is the larger opportunity.
