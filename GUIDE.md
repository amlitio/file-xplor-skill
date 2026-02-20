# Xplor — Complete Guide

## What You Have & What Still Needs Building

### Status Dashboard

| Component | Status | Files Exist | Deployable |
|-----------|--------|-------------|------------|
| **Claude AI Skill** (specification) | ✅ Complete | SKILL.md + 9 reference docs | Yes — upload to Claude Project |
| **Web App: Document Mode** (PDF → graph) | 🟡 Partial | Explorer.js, ShareButton.js, NetworkChat.js, 3 API routes | Need remaining components |
| **Web App: Existing Deployment** | ✅ Live | file-xplor.vercel.app | Already deployed on Vercel |
| **CLI Package** | ❌ Specs only | cli-spec.md has full design | Needs code written |
| **Code Ingestion Pipeline** | ❌ Specs only | code-mode.md has full design | Needs code written |
| **Skill Graph Pipeline** | ❌ Specs only | skill-graph-spec.md has full design | Needs code written |
| **MCP Server** | ❌ Specs only | mcp-server-spec.md has full design | Needs code written |
| **Agent Intelligence Layer** | ❌ Specs only | agent-intelligence.md has full design | Needs code written |

### What Each Piece Does

**The Skill (specs)** = A detailed blueprint that teaches Claude how to build Xplor.
When you give Claude this skill, it can scaffold the entire platform from scratch.

**The Web App (code)** = The actual running application at file-xplor.vercel.app.
Document Mode works today. Code Mode and Skill Graph Mode need to be added.

**The CLI** = A terminal tool (`npm install -g xplor`) for developers.
Indexes repos, starts MCP servers, manages skill graphs. Not yet built.

---

## Part 1: Using Xplor as a Claude Skill

This is the fastest path to value. Upload the skill files and Claude becomes an
Xplor expert — it can build any part of the platform for you on demand.

### Method A: Claude Project (Recommended)

This gives you a dedicated workspace where Claude always has Xplor context.

**Step 1:** Go to https://claude.ai and click **Projects** in the left sidebar.

**Step 2:** Click **"Create a project"** (or the + button).

**Step 3:** Name it: `Xplor Engine`

**Step 4:** In the project, click **"Add content"** → **"Upload files"**

**Step 5:** Upload these files (all from the xplor-skill folder):

```
Required (upload all of these):
├── SKILL.md
├── references/skill-graph-spec.md
├── references/agent-intelligence.md
├── references/code-mode.md
├── references/cli-spec.md
├── references/mcp-server-spec.md
├── references/document-mode.md
├── references/explorer-architecture.md
├── references/design-system.md
└── references/api-contracts.md
```

**Step 6:** In the **"Custom Instructions"** field for the project, paste:

```
You are building Xplor, a structured cognition engine. Use the uploaded
skill files as your blueprint. When I ask you to build any component,
reference the appropriate spec file and write production-ready code.

Architecture: Next.js 14 (App Router) + Firebase + Claude API + Tree-sitter.
Design: Dark theme (#0A0A0F), gradient (#FF6B6B → #4ECDC4), Space Grotesk + Outfit.
Three modes: Document (PDF → entities), Code (AST → call chains), Skill Graph (markdown → knowledge graph).

Always write complete, deployable files. Follow the design system exactly.
```

**Step 7:** Start a conversation in this project.

### What You Can Now Ask Claude

Once the skill is loaded, Claude can build any piece. Example prompts:

**Document Mode (extending what's live):**
- "Add a code upload mode to the existing UploadScreen — accept GitHub URLs and ZIP files alongside PDFs"
- "Build the trending page that shows the most-viewed shared projects"
- "Add the LandingPage component with pricing tiers"

**Code Mode (new):**
- "Build the code ingestion pipeline — file discovery, AST parsing with Tree-sitter, entity and relationship extraction"
- "Create the knowledge-graph.js core with in-memory graph, search, and traversal methods"
- "Write the xplor index CLI command that indexes a repo and saves the graph"

**Skill Graph Mode (new):**
- "Build the skill graph pipeline — wikilink parser, YAML frontmatter extractor, MOC detection"
- "Create the SkillGraphTraverser class with 5-level progressive disclosure"
- "Build the attention scoring system from agent-intelligence.md"
- "Create the xplor skill init command that scaffolds a new skill graph"

**MCP Server (new):**
- "Build the MCP server with all 8 tools — query, context, impact, callers, callees, search, traverse, scan"
- "Show me the Claude Code configuration to connect to the Xplor MCP server"

**Integration:**
- "Build the multi-domain fusion system that merges code + docs + skill graphs"
- "Add the traversal telemetry UI to the Explorer — show a heatmap of loaded nodes"
- "Create the graph-level evaluation framework to benchmark skill graphs vs flat prompts"

### Method B: Custom Instructions (Simpler, Less Powerful)

If you don't want a full project, add this to claude.ai → Settings → Profile:

```
I'm building Xplor, a structured cognition engine with three modes:
Document (PDF → entity graphs), Code (AST → call chain graphs), and
Skill Graph (markdown + wikilinks → knowledge graphs for AI agents).
Stack: Next.js 14, Firebase, Claude API, Tree-sitter, MCP SDK.
Design: dark theme #0A0A0F, gradients #FF6B6B → #4ECDC4.
When I ask about Xplor, help me build production-ready components.
```

This is lighter — Claude won't have the detailed specs in context, but it'll
understand the high-level architecture.

### Method C: Claude Code (Terminal)

If you use Claude Code for development:

```bash
# In your xplor project directory
mkdir -p .claude/skills/xplor
cp -r xplor-skill/* .claude/skills/xplor/
```

Claude Code will auto-detect and reference the skill during coding sessions.

---

## Part 2: Deploying the Web App

### What's Already Live

**file-xplor.vercel.app** is your existing deployment with:
- Firebase Auth (Google + Email)
- PDF upload → AI entity extraction
- Knowledge graph Explorer
- Save projects to Firestore
- Stripe Pro subscription ($9.99/mo)

The code lives at: https://github.com/amlitio/file-xplor

### Adding the New Features to the Live App

To extend the live app with Code Mode and Skill Graph Mode, you'd work
in the existing file-xplor repo:

**Step 1: Clone your repo**
```bash
git clone https://github.com/amlitio/file-xplor.git
cd file-xplor
npm install
```

**Step 2: Install new dependencies**
```bash
npm install web-tree-sitter gray-matter @modelcontextprotocol/sdk
```

**Step 3: Add the new files we built in previous sessions**

Copy these into your project (you have these from our previous downloads):

```
components/Explorer.js          ← Updated with ShareButton + NetworkChat
components/ShareButton.js       ← Social sharing modal
components/NetworkChat.js       ← AI chat panel (Pro feature)
app/api/share/route.js          ← Share CRUD endpoint
app/api/chat/route.js           ← Chat endpoint
app/api/trending/route.js       ← Trending endpoint
app/shared/[id]/page.js         ← Public shared view
```

**Step 4: Build Code Mode & Skill Graph Mode**

This is where the Claude skill earns its keep. Open your Xplor Claude Project
and ask Claude to build each piece:

```
"Using the code-mode.md spec, build the full code ingestion pipeline.
I need: file-discovery.js, ast-parser.js, entity-extractor.js, and
relationship-extractor.js. Put them in src/ingestion/."
```

```
"Using the skill-graph-spec.md, build the skill graph pipeline.
I need: wikilink-parser.js, skill-graph-builder.js, and
skill-graph-traverser.js. Put them in src/skill-graph/."
```

```
"Build a CodeUploadScreen component that accepts GitHub URLs and ZIP files.
Match the dark theme design system from design-system.md."
```

```
"Build a SkillGraphUploadScreen component that accepts a folder of markdown
files, parses them, and sends them to the Explorer."
```

**Step 5: Update the page router**

In `app/page.js`, add mode selection so users can choose:
- 📄 Analyze Documents (existing)
- 💻 Analyze Code (new)
- 🧠 Explore Skill Graph (new)

**Step 6: Deploy**
```bash
git add .
git commit -m "Add Code Mode + Skill Graph Mode"
git push origin main
```

Vercel auto-deploys from your main branch.

### New Environment Variables Needed

Add to your Vercel dashboard (Settings → Environment Variables):

```
# Existing (already set)
NEXT_PUBLIC_FIREBASE_API_KEY=...
ANTHROPIC_API_KEY=...
STRIPE_SECRET_KEY=...

# No new env vars needed for Code Mode (Tree-sitter runs client-side)
# No new env vars needed for Skill Graph Mode (parsing is local)
```

---

## Part 3: Building the CLI (Separate Package)

The CLI is a separate npm package — not part of the web app.

**Step 1: Create a new repo**
```bash
mkdir xplor-cli
cd xplor-cli
npm init -y
```

**Step 2: Ask Claude to build it**

In your Xplor Claude Project:

```
"Using cli-spec.md, build the complete CLI package. Start with:
1. bin/xplor.js entry point with commander.js
2. src/commands/index.js (xplor index command)
3. src/ingestion/pipeline.js (code ingestion)
4. src/graph/knowledge-graph.js (in-memory graph)

Make it installable with npm install -g."
```

**Step 3: Build the MCP server**

```
"Using mcp-server-spec.md, build src/mcp/server.js with all 8 tools.
Use @modelcontextprotocol/sdk. Include the traverse and scan tools
for skill graphs."
```

**Step 4: Add skill graph commands**

```
"Build the xplor skill subcommands: init, validate, stats, viz.
Use the skill-graph-spec.md for the validation rules and stats output format."
```

**Step 5: Test locally**
```bash
npm link          # Makes 'xplor' command available globally
xplor index .     # Test indexing the CLI project itself
xplor mcp         # Test MCP server
```

**Step 6: Publish to npm**
```bash
npm login
npm publish
```

Now anyone can `npm install -g xplor` and use it.

---

## Part 4: Recommended Build Order

If you want to build everything, here's the priority order:

### Sprint 1: Skill as Claude Project (30 minutes)
- Upload skill files to Claude Project ← **do this first**
- Test by asking Claude to build components
- Immediate value: Claude becomes your Xplor expert

### Sprint 2: Web App Code Mode (2-3 hours with Claude)
- Ask Claude (in your Xplor project) to build the code ingestion pipeline
- Add CodeUploadScreen component
- Add mode selector to app/page.js
- Deploy to Vercel

### Sprint 3: Web App Skill Graph Mode (2-3 hours with Claude)
- Ask Claude to build the skill graph pipeline
- Add SkillGraphUploadScreen
- Update Explorer to handle skill graph entity types + colors
- Deploy

### Sprint 4: CLI + MCP Server (3-4 hours with Claude)
- Ask Claude to build the CLI package
- Test with real repos
- Build MCP server
- Test with Claude Code
- Publish to npm

### Sprint 5: Agent Intelligence (2-3 hours with Claude)
- Attention scoring
- Traversal telemetry
- Telemetry visualization in Explorer
- Multi-domain fusion

### Sprint 6: Polish & Growth (ongoing)
- Landing page with all three modes showcased
- Public skill graph gallery (like trending, but for knowledge graphs)
- Blog post / manifesto for the vision
- Example skill graphs (therapy, trading, legal, company)

---

## Part 5: Quick Reference

### Skill Files → What They Build

| When you want to build... | Ask Claude to reference... |
|--------------------------|--------------------------|
| PDF upload + entity extraction | document-mode.md |
| AST parsing + code graphs | code-mode.md |
| Wikilink parsing + skill graphs | skill-graph-spec.md |
| CLI commands | cli-spec.md |
| MCP server for AI agents | mcp-server-spec.md |
| Attention scoring + telemetry | agent-intelligence.md |
| Explorer graph visualization | explorer-architecture.md |
| UI colors + typography + patterns | design-system.md |
| API request/response formats | api-contracts.md |

### Key Commands (once CLI is built)

```bash
# Document Mode (web only)
→ Upload PDFs at file-xplor.vercel.app

# Code Mode
xplor index ./my-repo
xplor query "find auth functions"
xplor impact validateToken

# Skill Graph Mode
xplor skill init my-domain
xplor skill validate ./my-domain
xplor skill stats ./my-domain

# MCP Server (for AI agents)
xplor mcp

# Multi-Domain Fusion
xplor fuse --code ./repo --skills ./practices --output merged

# Web UI
xplor serve --open
```

### GitHub Repos

| Repo | Purpose |
|------|---------|
| `github.com/amlitio/file-xplor` | Web app (Next.js, deployed on Vercel) |
| `github.com/amlitio/file-xplor-skill` | Claude AI Skill (specs + reference docs) |
| `github.com/amlitio/xplor-cli` (future) | CLI + MCP server (npm package) |

---

## Summary

**The skill is your accelerator.** You don't need to write 10,000 lines of code
by hand. Upload the skill to a Claude Project, and Claude can generate any
component on demand — following the exact architecture, design system, and API
contracts you've defined.

**The web app is your product.** Document Mode is live today. Code Mode and Skill
Graph Mode are the next two sprints. Each one is 2-3 hours of Claude-assisted
development.

**The CLI is your developer play.** Once the web app covers all three modes, extract
the ingestion pipelines into a CLI package and publish to npm. The MCP server turns
every Xplor graph into an AI agent tool.

**The vision is the moat.** Skill graphs — structured knowledge systems that AI
agents can traverse — is something nobody else is doing at this level. Code is one
domain. Structured cognition is the platform.
