# Agent Intelligence Layer — Specification

## Overview

The Agent Intelligence Layer sits between the raw knowledge graph and the interfaces
(web UI, CLI, MCP). It makes the graph **smart** — deciding what to load, when to load
it, and how to present it to agents.

This is what separates Xplor from a search engine. Search finds things. The intelligence
layer **understands what the agent needs and delivers it efficiently**.

---

## 1. Traversal Heuristics

### Attention Scoring

Every node gets a dynamic attention score based on the current query context.
Higher scores mean more likely to be relevant and loaded.

```javascript
function computeAttentionScore(node, query, graph) {
  let score = 0;

  // Semantic relevance to query (0-40 points)
  score += semanticSimilarity(node.description + " " + node.name, query) * 40;

  // Graph centrality — well-connected nodes are more likely useful (0-20 points)
  const inDegree = graph.edges.filter((e) => e.target === node.id).length;
  const outDegree = graph.edges.filter((e) => e.source === node.id).length;
  score += Math.min(20, (inDegree + outDegree) * 2);

  // Type bonus — MOCs get priority as navigation aids (0-15 points)
  if (node.type === "moc") score += 15;
  if (node.type === "framework") score += 10;
  if (node.type === "technique") score += 5;

  // Recency bonus — recently updated nodes may be more relevant (0-10 points)
  if (node.updatedAt) {
    const daysSinceUpdate = (Date.now() - new Date(node.updatedAt)) / 86400000;
    score += Math.max(0, 10 - daysSinceUpdate * 0.1);
  }

  // Domain match — if query mentions a domain, boost matching nodes (0-15 points)
  const queryDomains = extractDomains(query);
  if (queryDomains.includes(node.domain)) score += 15;

  return Math.min(100, score);
}
```

### Path Ranking

When traversing from node A, rank outgoing links by expected utility:

```javascript
function rankPaths(currentNode, graph, query, visited) {
  const outLinks = graph.edges.filter((e) => e.source === currentNode.id);

  return outLinks
    .map((link) => {
      const targetNode = graph.nodes.find((n) => n.id === link.target);
      if (!targetNode || visited.has(link.target)) return null;

      return {
        link,
        targetNode,
        score: computeAttentionScore(targetNode, query, graph),
        contextRelevance: semanticSimilarity(link.context, query),
        isNew: !visited.has(link.target),
      };
    })
    .filter(Boolean)
    .sort((a, b) => b.score + b.contextRelevance - a.score - a.contextRelevance);
}
```

### Progressive Disclosure Control

Automatically determine how deep to load based on graph size and query specificity:

```javascript
function determineLoadDepth(graph, query) {
  const graphSize = graph.nodes.length;
  const querySpecificity = estimateSpecificity(query); // 0 = vague, 1 = specific

  // Small graphs (< 20 nodes): load more freely
  if (graphSize < 20) return { maxNodes: 10, maxDepth: 4 };

  // Medium graphs (20-100): balance breadth and depth
  if (graphSize < 100) {
    if (querySpecificity > 0.7) return { maxNodes: 8, maxDepth: 3 };
    return { maxNodes: 12, maxDepth: 2 };
  }

  // Large graphs (100+): be selective
  if (querySpecificity > 0.7) return { maxNodes: 6, maxDepth: 3 };
  return { maxNodes: 10, maxDepth: 2 };
}
```

---

## 2. Context Injection

When an agent needs context for a task, the intelligence layer assembles the
optimal context window from the graph.

### Context Assembly Algorithm

```javascript
async function assembleContext(graph, task, maxTokens = 8000) {
  const traverser = new SkillGraphTraverser(graph);

  // 1. Start from most relevant MOC
  const mocs = traverser.getIndex();
  const bestMoc = mocs.reduce((best, moc) =>
    semanticSimilarity(moc.description, task) >
    semanticSimilarity(best.description, task) ? moc : best
  );

  // 2. Scan neighborhood
  const candidates = traverser.scan(task);

  // 3. Score all candidates
  const scored = candidates.map((c) => ({
    ...c,
    score: computeAttentionScore(c, task, graph),
  })).sort((a, b) => b.score - a.score);

  // 4. Greedily add nodes until token budget exhausted
  const context = [];
  let tokensUsed = 0;

  for (const candidate of scored) {
    const node = traverser.load(candidate.id);
    const nodeTokens = estimateTokens(node.content);

    if (tokensUsed + nodeTokens > maxTokens) {
      // Try sections only
      const sections = traverser.getSections(candidate.id);
      const sectionTokens = estimateTokens(sections.join("\n"));
      if (tokensUsed + sectionTokens <= maxTokens) {
        context.push({ ...candidate, content: sections.join("\n"), level: "sections" });
        tokensUsed += sectionTokens;
      }
      continue;
    }

    context.push({ ...candidate, content: node.content, level: "full" });
    tokensUsed += nodeTokens;
  }

  return {
    context,
    tokensUsed,
    nodesLoaded: context.length,
    entryPoint: bestMoc,
    telemetry: traverser.getTelemetry(),
  };
}
```

### Context Format for Agents

When injecting context into an agent's prompt, structure it clearly:

```markdown
## Xplor Context: [domain]

### Entry Point: [MOC name]
[MOC description]

### Loaded Knowledge Nodes

#### [Node 1 Name] (technique)
[Full or sectioned content]

#### [Node 2 Name] (framework)
[Full or sectioned content]

### Related but Unloaded
- [Node 3] — [description only, available if needed]
- [Node 4] — [description only]

### Graph Context
- Total nodes in domain: N
- Nodes loaded: M
- Coverage: M/N (X%)
```

---

## 3. Traversal Telemetry

Let users see exactly what the agent loaded and why. This makes skill graphs
**auditable** — you can see if the agent is missing context or loading irrelevant nodes.

### Telemetry Schema

```javascript
{
  sessionId: "sess-abc123",
  timestamp: "2025-02-19T...",
  query: "How to help a client with catastrophizing",
  
  traversal: {
    entryPoint: "cbt-techniques",
    nodesVisited: 12,      // Seen at any disclosure level
    nodesLoaded: 5,        // Fully loaded into context
    nodesSkipped: 7,       // Visited but not loaded
    maxDepthReached: 3,
    totalTokensInjected: 4200,
  },
  
  loadLog: [
    {
      nodeId: "cognitive-distortions",
      level: "full",            // full | sections | description
      score: 87,
      reason: "High semantic match + high in-degree",
      timestamp: 1708360000000,
    },
    {
      nodeId: "cognitive-reframing",
      level: "full",
      score: 82,
      reason: "Direct link from cognitive-distortions + query match",
      timestamp: 1708360001000,
    },
    {
      nodeId: "thought-records",
      level: "sections",
      score: 64,
      reason: "Referenced by cognitive-reframing, partial load for token budget",
      timestamp: 1708360002000,
    },
  ],
  
  skipLog: [
    {
      nodeId: "behavioral-activation",
      score: 31,
      reason: "Low semantic relevance to catastrophizing query",
    },
    {
      nodeId: "exposure-hierarchy",
      score: 28,
      reason: "Behavioral technique, not cognitive — below threshold",
    },
  ],
  
  recommendations: [
    "Consider loading 'socratic-questioning' — referenced by 2 loaded nodes",
    "Node 'validation-first' may be relevant — mentioned as contraindication",
  ],
}
```

### Web UI Telemetry Visualization

The Explorer shows telemetry as:

1. **Heatmap overlay** on the graph — loaded nodes glow bright, visited nodes dim,
   unvisited nodes are dark
2. **Load timeline** — sidebar showing the order nodes were loaded
3. **Coverage meter** — percentage of graph loaded for this query
4. **Skip reasoning** — hover over unloaded nodes to see why they were skipped

---

## 4. Multi-Domain Fusion

The most powerful capability: merge multiple graphs into a single queryable layer.

### Use Cases

- **Code + Docs**: Merge a codebase graph with company documentation. "What does the
  auth middleware do and what's our security policy for token expiry?"
- **Code + Skills**: Merge a codebase graph with coding best practices. Agent gets
  both the code context AND the patterns to apply.
- **Company Knowledge**: Merge org structure + product docs + process docs + culture
  norms. Agent has complete company context.
- **Legal + Product**: Merge contract patterns with product features for compliance
  analysis.

### Fusion Algorithm

```javascript
function fuseGraphs(graphs) {
  const fused = {
    nodes: [],
    edges: [],
    domains: [],
  };

  const nodeIndex = new Map();

  for (const graph of graphs) {
    const domainPrefix = graph.domain || graph.name;
    fused.domains.push(domainPrefix);

    // Add nodes with domain prefix to avoid ID collisions
    for (const node of graph.nodes) {
      const fusedId = `${domainPrefix}:${node.id}`;
      const fusedNode = { ...node, id: fusedId, domain: domainPrefix, originalId: node.id };
      fused.nodes.push(fusedNode);
      nodeIndex.set(fusedId, fusedNode);
      nodeIndex.set(node.name?.toLowerCase(), fusedNode); // Name-based lookup
    }

    // Add edges with updated IDs
    for (const edge of graph.edges) {
      fused.edges.push({
        ...edge,
        source: `${domainPrefix}:${edge.source}`,
        target: `${domainPrefix}:${edge.target}`,
        domain: domainPrefix,
      });
    }
  }

  // Cross-domain linking: find nodes with matching names across domains
  const nameGroups = {};
  fused.nodes.forEach((n) => {
    const key = n.name?.toLowerCase();
    if (key) {
      if (!nameGroups[key]) nameGroups[key] = [];
      nameGroups[key].push(n);
    }
  });

  for (const [name, nodes] of Object.entries(nameGroups)) {
    if (nodes.length > 1) {
      // Create CROSS_DOMAIN edges between same-named entities
      for (let i = 0; i < nodes.length; i++) {
        for (let j = i + 1; j < nodes.length; j++) {
          fused.edges.push({
            source: nodes[i].id,
            target: nodes[j].id,
            type: "CROSS_DOMAIN",
            label: `shared: ${name}`,
          });
        }
      }
    }
  }

  return fused;
}
```

### Fusion in the Web UI

The Explorer shows fused graphs with:
- **Domain-colored regions** — nodes from different graphs cluster by domain
- **Cross-domain edges** highlighted with a distinct style (thick dashed line)
- **Domain filter toggle** — show/hide specific domains
- **Fusion search** — query across all domains simultaneously

### Fusion via CLI

```bash
# Fuse a code graph with a skill graph
xplor fuse --code ./my-repo --skills ./coding-best-practices --output merged

# Fuse multiple skill graphs
xplor fuse --skills ./therapy-cbt --skills ./therapy-attachment --output therapy-full

# Fuse code + docs + skills
xplor fuse --code ./my-repo --docs ./company-wiki --skills ./eng-practices --output full-context
```

---

## 5. Graph-Level Evaluation

Measure how well agents perform with structured skill graphs vs flat prompts.

### Benchmark Protocol

```
1. Define a set of tasks that require domain knowledge
2. Run each task in three configurations:
   a. No context (baseline)
   b. Flat skill file (single large document)
   c. Skill graph with traversal (Xplor)
3. Grade outputs on: accuracy, completeness, depth of reasoning
4. Compare scores and context efficiency (tokens used)
```

### Metrics

| Metric | What It Measures |
|--------|-----------------|
| **Accuracy** | Correctness of agent's output |
| **Completeness** | Coverage of relevant domain concepts |
| **Reasoning Depth** | Evidence of cross-concept connections |
| **Context Efficiency** | Quality per token of context injected |
| **Traversal Quality** | Did the agent load the right nodes? |
| **Coverage** | What percentage of relevant nodes were loaded? |

### Expected Results

Skill graphs should outperform flat files on:
- Complex, multi-step reasoning tasks
- Tasks requiring cross-concept connections
- Large domain knowledge bases (where flat files exceed context limits)
- Tasks where different subtopics have varying relevance

Flat files may win on:
- Simple, single-concept queries
- Very small domains (< 10 concepts)
- Tasks where all knowledge is equally relevant
