# Skill Graph API — Web Contracts

All routes: `app/api/skillgraph/*/route.js` (Next.js App Router)

---

## POST /api/skillgraph/ingest

Upload a ZIP or GitHub URL, parse all markdown files, build skill graph.

### Request
```json
{
  "sourceType": "zip" | "github",
  "source": "<file upload>" | "https://github.com/user/repo",
  "branch": "main",
  "subdirectory": "skills/"
}
```

For ZIP: multipart/form-data with `file` field + `sourceType` field.
For GitHub: JSON body.

### Response
```json
{
  "graphId": "therapy-cbt",
  "kind": "skill",
  "metrics": {
    "nodeCount": 24,
    "edgeCount": 67,
    "density": 0.12,
    "domains": ["therapy"],
    "typeBreakdown": { "technique": 12, "moc": 3, "claim": 4, "framework": 3, "exploration": 2 },
    "clusterCount": 4,
    "orphanCount": 0
  },
  "validation": {
    "score": 92,
    "brokenLinks": [
      { "source": "cognitive-reframing.md", "target": "thought-journals", "context": "the [[thought-journals]] worksheet" }
    ],
    "missingDescriptions": ["grounding-techniques.md"],
    "orphans": [],
    "circularOnly": []
  }
}
```

### Limits
- Max ZIP size: 10MB
- Max file count: 500 files
- Max total markdown: 5MB
- Rate: 20 ingests/hour/user

### Errors
- `400` — Invalid source type or empty archive
- `413` — Exceeds size limits
- `422` — No .md files found
- `429` — Rate limited

---

## POST /api/skillgraph/scan

Frontmatter-only search. Returns Level 1 data — **never** returns markdown body.

### Request
```json
{
  "graphId": "therapy-cbt",
  "query": "emotional regulation",
  "domain": "therapy",
  "limit": 20
}
```

### Response
```json
{
  "results": [
    {
      "id": "skill:emotional-regulation",
      "name": "Emotional Regulation",
      "type": "technique",
      "domain": "therapy",
      "description": "Core prerequisite skill for cognitive restructuring work...",
      "tags": ["foundational", "prerequisite"],
      "inDegree": 8,
      "outDegree": 4
    }
  ],
  "total": 3,
  "level": 1
}
```

**Enforcement:** Response must NEVER include `content.full` or `content.sections`.

---

## POST /api/skillgraph/traverse

Navigate the graph from a start node following relevant links. Returns progressive
disclosure content at the appropriate level per node.

### Request
```json
{
  "graphId": "therapy-cbt",
  "startNode": "skill:cbt-techniques",
  "query": "how to help a client with catastrophizing",
  "maxDepth": 3,
  "maxNodes": 8,
  "tokenBudget": 6000
}
```

### Response
```json
{
  "entryPoint": {
    "id": "skill:cbt-techniques",
    "name": "CBT Techniques",
    "type": "moc",
    "description": "..."
  },
  "path": [
    {
      "id": "skill:cognitive-distortions",
      "name": "Cognitive Distortions",
      "level": 4,
      "content": "Full markdown content...",
      "score": 87,
      "reason": "High semantic match + highest in-degree in cluster"
    },
    {
      "id": "skill:cognitive-reframing",
      "name": "Cognitive Reframing",
      "level": 4,
      "content": "Full markdown content...",
      "score": 82,
      "reason": "Direct link from cognitive-distortions + query keyword match"
    },
    {
      "id": "skill:thought-records",
      "name": "Thought Records",
      "level": 3,
      "content": "## Application\nThe primary tool is...\n## When to Use\n...",
      "score": 64,
      "reason": "Referenced by cognitive-reframing, sections-only for token budget"
    }
  ],
  "unloaded": [
    { "id": "skill:behavioral-activation", "score": 31, "reason": "Low relevance to catastrophizing" },
    { "id": "skill:exposure-hierarchy", "score": 28, "reason": "Behavioral, not cognitive" }
  ],
  "telemetry": {
    "nodesVisited": 12,
    "nodesLoaded": 5,
    "nodesSkipped": 7,
    "tokensUsed": 4200,
    "tokenBudget": 6000,
    "coveragePercent": 20.8,
    "maxDepthReached": 3
  }
}
```

**Enforcement:** Nodes at Level 3 must NOT include `content.full`. Nodes at Level 1
must NOT include any content. The `tokenBudget` is a hard ceiling — stop loading
before exceeding it.

---

## POST /api/skillgraph/context

Assemble the optimal context pack for a given query. Used by MCP server and chat.
This is the main "give me what I need to know" endpoint.

### Request
```json
{
  "graphId": "therapy-cbt",
  "query": "How should I approach a client showing signs of catastrophizing?",
  "tokenBudget": 6000
}
```

### Response
```json
{
  "contextPack": {
    "entryPoint": "skill:cbt-techniques",
    "nodes": [
      { "id": "skill:cognitive-distortions", "level": 4, "content": "...", "tokens": 1200 },
      { "id": "skill:cognitive-reframing", "level": 4, "content": "...", "tokens": 1400 },
      { "id": "skill:thought-records", "level": 3, "content": "...", "tokens": 600 },
      { "id": "skill:socratic-questioning", "level": 3, "content": "...", "tokens": 500 },
      { "id": "skill:validation-first", "level": 1, "content": null, "tokens": 0, "description": "When the thought is accurate, not distorted" }
    ],
    "totalTokens": 3700,
    "tokenBudget": 6000
  },
  "telemetry": { ... }
}
```

---

## POST /api/skillgraph/validate

Run quality validation on a skill graph.

### Request
```json
{
  "graphId": "therapy-cbt"
}
```

### Response
```json
{
  "score": 92,
  "maxScore": 100,
  "issues": {
    "brokenLinks": [
      { "source": "cognitive-reframing.md", "target": "thought-journals", "penalty": -10 }
    ],
    "missingDescriptions": [
      { "file": "grounding-techniques.md", "penalty": -5 }
    ],
    "missingTypes": [],
    "orphans": [],
    "circularOnly": []
  },
  "bonuses": {
    "mocCoverage": 8,
    "linkDensityHealth": 9
  },
  "summary": "2 broken links and 1 missing description. Fix these to reach 100."
}
```

### Scoring Rubric

See `references/skill-graph-quality.md` for the full rubric.

---

## GET /api/skillgraph/stats?graphId=therapy-cbt

### Response
```json
{
  "graphId": "therapy-cbt",
  "kind": "skill",
  "nodeCount": 24,
  "edgeCount": 67,
  "density": 0.12,
  "avgDegree": 5.6,
  "maxInDegree": { "nodeId": "skill:cognitive-distortions", "value": 11 },
  "maxOutDegree": { "nodeId": "skill:index", "value": 8 },
  "clusterCount": 4,
  "orphanCount": 0,
  "domains": ["therapy"],
  "typeBreakdown": { "technique": 12, "moc": 3, "claim": 4, "framework": 3, "exploration": 2 },
  "edgeTypeBreakdown": { "REFERENCES": 52, "CLUSTERS": 12, "EXTENDS": 3 }
}
```
