# CLI Specification

## Install

```bash
npm install -g xplor
```

## Commands

### Core Commands

```bash
xplor index <path>               # Index a repo (local path, GitHub URL, or ZIP)
xplor index ./repo --languages js,ts  # Filter by language
xplor mcp                        # Start MCP server (stdio transport)
xplor mcp --transport sse --port 3100  # SSE transport
xplor serve                      # Start web UI locally
xplor serve --port 3000 --open   # Custom port, auto-open browser
xplor query "find auth functions"  # Query the graph
xplor impact src/auth/validate.ts  # Downstream impact analysis
xplor callers validateToken      # Upstream call chain
```

### Skill Graph Commands

```bash
xplor skill init <name>          # Scaffold a new skill graph
xplor skill validate <path>      # Check for broken links, missing frontmatter
xplor skill stats <path>         # Graph topology analysis
xplor skill viz <path> --open    # Visual graph in browser
```

### Fusion Commands

```bash
xplor fuse --code ./repo --skills ./best-practices --output merged
xplor fuse --skills ./cbt --skills ./attachment --output therapy-full
xplor fuse --code ./repo --docs ./wiki --skills ./practices --output full
```

## Project Structure

```
xplor-cli/
├── package.json
├── bin/xplor.js                    # Entry point (#!/usr/bin/env node)
├── src/
│   ├── commands/
│   │   ├── index.js                # xplor index
│   │   ├── mcp.js                  # xplor mcp
│   │   ├── serve.js                # xplor serve
│   │   ├── query.js                # xplor query
│   │   ├── impact.js               # xplor impact
│   │   ├── skill-init.js           # xplor skill init
│   │   ├── skill-validate.js       # xplor skill validate
│   │   ├── skill-stats.js          # xplor skill stats
│   │   └── fuse.js                 # xplor fuse
│   ├── ingestion/
│   │   ├── pipeline.js             # Code ingestion orchestrator
│   │   ├── skill-graph-pipeline.js # Skill graph ingestion
│   │   ├── file-discovery.js
│   │   ├── ast-parser.js
│   │   ├── entity-extractor.js
│   │   └── wikilink-parser.js
│   ├── graph/
│   │   ├── knowledge-graph.js      # In-memory graph store
│   │   ├── traverser.js            # Traversal engine
│   │   ├── attention.js            # Attention scoring
│   │   └── fusion.js               # Multi-domain fusion
│   └── mcp/
│       └── server.js               # MCP server
├── grammars/                       # Tree-sitter WASM files
└── templates/                      # Skill graph scaffolding templates
```

## Entry Point

```javascript
#!/usr/bin/env node
import { Command } from "commander";

const program = new Command();
program.name("xplor").description("Structured cognition engine").version("1.0.0");

program.command("index <path>")
  .option("-l, --languages <langs>", "Language filter")
  .option("-v, --verbose", "Detailed output")
  .action(async (p, o) => (await import("../src/commands/index.js")).run(p, o));

program.command("mcp")
  .option("-g, --graph <n>", "Specific graph")
  .option("-t, --transport <type>", "stdio or sse", "stdio")
  .option("-p, --port <n>", "SSE port", "3100")
  .action(async (o) => (await import("../src/commands/mcp.js")).run(o));

program.command("serve")
  .option("-p, --port <n>", "Port", "8080")
  .option("-o, --open", "Open browser")
  .action(async (o) => (await import("../src/commands/serve.js")).run(o));

program.command("query <question>")
  .option("-g, --graph <n>", "Specific graph")
  .action(async (q, o) => (await import("../src/commands/query.js")).run(q, o));

program.command("impact <target>")
  .action(async (t, o) => (await import("../src/commands/impact.js")).run(t, o));

const skill = program.command("skill").description("Skill graph tools");
skill.command("init <name>").action(async (n) => (await import("../src/commands/skill-init.js")).run(n));
skill.command("validate <path>").action(async (p) => (await import("../src/commands/skill-validate.js")).run(p));
skill.command("stats <path>").action(async (p) => (await import("../src/commands/skill-stats.js")).run(p));
skill.command("viz <path>").option("-o, --open", "Open browser").action(async (p, o) => (await import("../src/commands/skill-viz.js")).run(p, o));

program.command("fuse")
  .option("--code <path>", "Code repo path")
  .option("--docs <path>", "Docs directory")
  .option("--skills <paths...>", "Skill graph paths")
  .option("--output <name>", "Output graph name")
  .action(async (o) => (await import("../src/commands/fuse.js")).run(o));

program.parse();
```

## package.json

```json
{
  "name": "xplor",
  "version": "1.0.0",
  "description": "Structured cognition engine — knowledge graphs for AI agents",
  "type": "module",
  "bin": { "xplor": "./bin/xplor.js" },
  "files": ["bin/", "src/", "grammars/", "templates/"],
  "dependencies": {
    "commander": "^12.0.0",
    "web-tree-sitter": "^0.22.0",
    "@anthropic-ai/sdk": "^0.30.0",
    "@modelcontextprotocol/sdk": "^1.0.0",
    "gray-matter": "^4.0.3",
    "chalk": "^5.3.0",
    "ora": "^8.0.0",
    "glob": "^10.0.0",
    "isomorphic-git": "^1.25.0",
    "express": "^4.18.0",
    "zod": "^3.22.0"
  }
}
```

## Graph Storage

```
~/.xplor/
├── config.json
└── graphs/
    ├── my-project.json        # Code graph
    ├── therapy-cbt.json       # Skill graph
    └── merged-full.json       # Fused graph
```
