# 🔮 Xplor — Structured Cognition Engine

> People underestimate the power of structured knowledge. Structured knowledge enables entirely new classes of applications.

**Xplor** is a Claude AI Skill that builds knowledge graph platforms transforming **documents**, **codebases**, and **structured knowledge systems** into traversable, AI-queryable intelligence graphs.

Like DeepWiki, but deeper. DeepWiki helps you understand code. Xplor lets you **analyze** it — because a knowledge graph tracks every relationship, not just descriptions.

---

## Three Modes

| Mode | Input | What You Get |
|------|-------|-------------|
| 📄 **Document** | PDFs, text files | People, organizations, locations, dates, concepts → interactive network graph |
| 💻 **Code** | Git repos, ZIP | Functions, classes, imports, call chains → CALLS/IMPORTS/EXTENDS/DEFINES graph |
| 🧠 **Skill Graph** | Markdown + wikilinks | Knowledge nodes, MOCs, claims, techniques → navigable intelligence for AI agents |

All three modes render through the same Explorer UI. They can be **fused** — code + docs + domain skills merged into a single queryable intelligence layer.

---

## What Makes Skill Graphs Different

Most AI skills are single files. One file, one capability. That works for simple tasks.

But real depth requires something structurally different.

A **skill graph** is a network of markdown files connected with `[[wikilinks]]`. Each file represents one complete thought, one technique, one methodology claim. The links between them create a traversable graph that AI agents navigate on demand.

```
therapy-cbt/
├── index.md                    ← Entry point MOC
├── techniques/
│   ├── cognitive-reframing.md  ← Links to [[thought-records]], [[cognitive-distortions]]
│   ├── thought-records.md      ← Links to [[socratic-questioning]]
│   └── exposure-hierarchy.md   ← Links to [[behavioral-activation]]
├── claims/
│   └── validation-first.md     ← Cross-links to [[grounding-techniques]]
└── frameworks/
    └── case-formulation.md     ← Integrates everything
```

The agent reads the index, understands the landscape, follows the relevant paths, and loads only what the current task requires. This is the difference between an agent that follows instructions and an agent that **understands a domain**.

---

## Features

### Core Platform
- 🔍 PDF upload → AI entity extraction (Claude API)
- 🕸️ Force-directed knowledge graph with zoom/pan
- 💾 Save/manage projects (Firebase)
- 🔗 Share system with public links
- 💬 AI chat about your documents
- 💳 Stripe Pro subscription

### Code Intelligence
- 🌳 AST parsing via Tree-sitter (JS, TS, Python)
- 📊 Call chain analysis, impact assessment
- 🤖 MCP server for Claude Code, Cursor, Claude Desktop
- ⌨️ CLI: `xplor index`, `xplor query`, `xplor impact`

### Skill Graphs
- 📝 Wikilink + YAML frontmatter parsing
- 🗺️ Maps of Content (MOCs) as navigation entry points
- 🧭 5-level progressive disclosure (index → descriptions → links → sections → full)
- ⚡ Attention scoring and path ranking
- 📊 Traversal telemetry (what loaded, why, what was skipped)
- 🔧 Authoring toolkit: init, validate, stats, viz

### Advanced
- 🔀 Multi-domain fusion (code + docs + skills merged)
- 📈 Graph-level evaluation (benchmark skill graphs vs flat prompts)
- 🎯 Context injection (assemble optimal context for any task)

---

## Installation

### Personal Use (Claude Projects)

1. Go to **claude.ai → Projects → Create Project**
2. Upload `SKILL.md` + all files from `references/` as project knowledge
3. Start conversations — Claude now builds Xplor-style platforms

### Public CLI

```bash
npm install -g xplor

xplor index ./my-repo              # Index a codebase
xplor skill init trading-strategy  # Scaffold a skill graph
xplor mcp                          # Start MCP server for AI agents
xplor serve                        # Launch web UI
```

### Claude Code Integration

```json
{
  "mcpServers": {
    "xplor": { "command": "xplor", "args": ["mcp"] }
  }
}
```

---

## Skill Structure

```
xplor-skill/
├── SKILL.md                           # Main skill (triggers + architecture)
├── references/
│   ├── document-mode.md               # PDF pipeline + Firestore schema
│   ├── code-mode.md                   # AST parsing + Tree-sitter
│   ├── skill-graph-spec.md            # Wikilinks, MOCs, traversal, authoring
│   ├── agent-intelligence.md          # Attention scoring, telemetry, fusion
│   ├── cli-spec.md                    # CLI commands + project structure
│   ├── mcp-server-spec.md             # MCP tools + agent config
│   ├── explorer-architecture.md       # Explorer component spec
│   ├── design-system.md               # Colors, typography, patterns
│   └── api-contracts.md               # API route schemas
└── evals/
    └── evals.json                     # Test cases
```

---

## Live Demo

Document Mode: **[file-xplor.vercel.app](https://file-xplor.vercel.app)**

---

## Vision

Xplor is positioned as:

- **A Knowledge Graph Engine** for code, documents, and structured knowledge
- **A Skill Graph Engine** for agent-navigable intelligence
- **"Git for Structured Knowledge"** — version, traverse, query, fuse
- **The Graph OS for AI agents** — infrastructure for domain-aware reasoning

Code is one domain. Structured knowledge is the larger opportunity.

---

## License

MIT

Built by [@amlitio](https://github.com/amlitio)
