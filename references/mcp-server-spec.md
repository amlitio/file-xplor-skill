# MCP Server Specification

## Overview

The Xplor MCP server exposes knowledge graphs to AI agents via the Model Context
Protocol. Works with code graphs, document graphs, and skill graphs.

## Tools

| Tool | Input | Purpose |
|------|-------|---------|
| `query` | natural language question | Find entities by description |
| `context` | symbol name or file path | Deep context + all relationships |
| `impact` | function/file name | Downstream dependency analysis |
| `callers` | function name + depth | Upstream call chain |
| `callees` | function name + depth | Downstream call chain |
| `search` | text query + limit | Full-text search across entities |
| `traverse` | start node + query + depth | Skill graph traversal with attention scoring |
| `scan` | query + domain | Scan frontmatter descriptions without loading content |

## Implementation

```javascript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

export async function startMcpServer(graph) {
  const server = new McpServer({ name: "xplor", version: "1.0.0" });

  server.tool("query", "Search the knowledge graph",
    { question: z.string() },
    async ({ question }) => {
      const results = graph.search(question);
      return { content: [{ type: "text", text: formatResults(results) }] };
    });

  server.tool("context", "Deep context for a symbol",
    { symbol: z.string() },
    async ({ symbol }) => {
      const entity = graph.findEntity(symbol);
      const rels = graph.getRelationships(entity.id);
      return { content: [{ type: "text", text: formatContext(entity, rels) }] };
    });

  server.tool("impact", "Downstream impact analysis",
    { target: z.string() },
    async ({ target }) => {
      const downstream = graph.traverseDownstream(target, ["CALLS", "IMPORTS"], 5);
      return { content: [{ type: "text", text: formatImpact(downstream) }] };
    });

  server.tool("callers", "Upstream call chain",
    { function: z.string(), depth: z.number().optional().default(3) },
    async ({ function: fn, depth }) => {
      const chain = graph.traverseUpstream(fn, "CALLS", depth);
      return { content: [{ type: "text", text: formatChain(chain) }] };
    });

  server.tool("callees", "Downstream call chain",
    { function: z.string(), depth: z.number().optional().default(3) },
    async ({ function: fn, depth }) => {
      const chain = graph.traverseDownstream(fn, ["CALLS"], depth);
      return { content: [{ type: "text", text: formatChain(chain) }] };
    });

  server.tool("search", "Full-text search",
    { query: z.string(), limit: z.number().optional().default(10) },
    async ({ query, limit }) => {
      const results = graph.fullTextSearch(query, limit);
      return { content: [{ type: "text", text: formatSearchResults(results) }] };
    });

  // --- SKILL GRAPH SPECIFIC ---

  server.tool("traverse", "Navigate a skill graph with attention-scored traversal",
    {
      startNode: z.string().describe("MOC or node ID to start from"),
      query: z.string().describe("What the agent is looking for"),
      maxDepth: z.number().optional().default(3),
      maxNodes: z.number().optional().default(8),
    },
    async ({ startNode, query, maxDepth, maxNodes }) => {
      const traverser = new SkillGraphTraverser(graph);
      const result = assembleContext(graph, query, {
        entryPoint: startNode, maxDepth, maxNodes,
      });
      return {
        content: [{
          type: "text",
          text: formatTraversalResult(result),
        }],
      };
    });

  server.tool("scan", "Scan node descriptions without loading full content",
    {
      query: z.string().describe("Topic to scan for"),
      domain: z.string().optional().describe("Filter by domain"),
    },
    async ({ query, domain }) => {
      const matches = graph.nodes
        .filter((n) => !domain || n.domain === domain)
        .filter((n) => matchesQuery(n, query))
        .map((n) => ({
          id: n.id, name: n.name,
          description: n.description,
          type: n.type, domain: n.domain,
        }));
      return {
        content: [{
          type: "text",
          text: matches.length
            ? matches.map((m) => `- **${m.name}** (${m.type}): ${m.description}`).join("\n")
            : "No matching nodes found.",
        }],
      };
    });

  const transport = new StdioServerTransport();
  await server.connect(transport);
}
```

## Agent Configuration

### Claude Code / Claude Desktop

`~/.claude/claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "xplor": { "command": "xplor", "args": ["mcp"] }
  }
}
```

### Cursor

Settings → MCP → Add:
```json
{
  "xplor": { "command": "xplor", "args": ["mcp"] }
}
```

## Graph API Interface

```typescript
interface KnowledgeGraph {
  nodes: Node[];
  edges: Edge[];
  search(query: string): Node[];
  findEntity(nameOrPath: string): Node | null;
  getCallers(entityId: string): Node[];
  getCallees(entityId: string): Node[];
  getRelationships(entityId: string, type?: string): Node[];
  traverseDownstream(target: string, edgeTypes: string[], maxDepth: number): TraversalResult[];
  traverseUpstream(target: string, edgeType: string, maxDepth: number): TraversalResult[];
  fullTextSearch(query: string, limit: number): Node[];
}
```
