# Skill Graph Quality — Scoring & Validation

## Quality Score

Every skill graph gets a score from 0-100. This score is deterministic —
same graph always produces same score.

### Penalty Rules (deductions from 100)

| Issue | Penalty | Detection |
|-------|---------|-----------|
| Broken wikilink (target file not found) | **-10** per link | Parse all `[[targets]]`, check against file index |
| Missing `description` in frontmatter | **-5** per file | Parse YAML, check for `description` key |
| Orphan node (no incoming or outgoing links) | **-3** per node | Check degree in assembled graph |
| Missing `type` in frontmatter | **-2** per file | Parse YAML, check for `type` key |
| Missing `domain` in frontmatter | **-1** per file | Parse YAML, check for `domain` key |
| Circular-only reference (A↔B with no other connections) | **-2** per pair | Detect nodes where all edges form closed pairs |

### Bonus Rules (additions, max +20)

| Quality Signal | Bonus | Detection |
|---------------|-------|-----------|
| MOC coverage | **+0 to +10** | `clusters_with_moc / total_clusters * 10` |
| Link density health | **+0 to +10** | Penalize dead ends, reward balanced degree distribution |

### Link Density Health Formula

```javascript
function linkDensityBonus(nodes, edges) {
  const degrees = nodes.map((n) => {
    const inD = edges.filter((e) => e.target === n.id).length;
    const outD = edges.filter((e) => e.source === n.id).length;
    return inD + outD;
  });

  const deadEnds = degrees.filter((d) => d <= 1).length;
  const deadEndRatio = deadEnds / nodes.length;

  // 0 dead ends = +10, >50% dead ends = +0
  return Math.round(Math.max(0, 10 * (1 - deadEndRatio * 2)));
}
```

### Score Calculation

```javascript
function calculateQualityScore(graph, validation) {
  let score = 100;

  // Penalties
  score -= validation.brokenLinks.length * 10;
  score -= validation.missingDescriptions.length * 5;
  score -= validation.orphans.length * 3;
  score -= validation.missingTypes.length * 2;
  score -= validation.missingDomains.length * 1;
  score -= validation.circularOnly.length * 2;

  // Floor at 0
  score = Math.max(0, score);

  // Bonuses (can't exceed 100)
  score += mocCoverageBonus(graph);
  score += linkDensityBonus(graph.nodes, graph.edges);

  return Math.min(100, score);
}
```

---

## Validation Report Format

```json
{
  "score": 92,
  "maxScore": 100,
  "issues": {
    "brokenLinks": [
      {
        "source": "cognitive-reframing.md",
        "target": "thought-journals",
        "context": "The primary tool is the [[thought-journals]] worksheet",
        "penalty": -10
      }
    ],
    "missingDescriptions": [
      { "file": "grounding-techniques.md", "penalty": -5 }
    ],
    "missingTypes": [],
    "missingDomains": [
      { "file": "case-formulation.md", "penalty": -1 }
    ],
    "orphans": [],
    "circularOnly": []
  },
  "bonuses": {
    "mocCoverage": { "value": 8, "clustersWithMoc": 3, "totalClusters": 4 },
    "linkDensityHealth": { "value": 9, "deadEnds": 1, "totalNodes": 24 }
  },
  "summary": "Score 92/100. Fix 1 broken link (-10) and 1 missing description (-5) to reach 100."
}
```

---

## Cluster Detection

### Algorithm: Connected Components (v1)

For v1, use simple connected components. Treat the graph as undirected for clustering.

```javascript
function detectClusters(nodes, edges) {
  const parent = {};
  nodes.forEach((n) => (parent[n.id] = n.id));

  function find(x) {
    if (parent[x] !== x) parent[x] = find(parent[x]);
    return parent[x];
  }

  function union(a, b) {
    const ra = find(a), rb = find(b);
    if (ra !== rb) parent[ra] = rb;
  }

  edges.forEach((e) => union(e.source, e.target));

  // Group nodes by root
  const clusters = {};
  nodes.forEach((n) => {
    const root = find(n.id);
    if (!clusters[root]) clusters[root] = [];
    clusters[root].push(n);
  });

  return Object.values(clusters)
    .filter((c) => c.length > 1)
    .sort((a, b) => b.length - a.length);
}
```

### MOC Coverage

A cluster "has MOC coverage" if at least one node of type `moc` exists in the cluster.

```javascript
function mocCoverageBonus(graph) {
  const clusters = detectClusters(graph.nodes, graph.edges);
  const clustersWithMoc = clusters.filter((c) =>
    c.some((n) => n.type === "moc")
  ).length;
  return Math.round((clustersWithMoc / Math.max(1, clusters.length)) * 10);
}
```

### Future (v2): Label Propagation

For larger graphs (100+ nodes), connected components is too coarse.
Upgrade to label propagation for semantic clustering:
- Initialize each node with unique label
- Iterate: each node adopts most frequent label among neighbors
- Converge when labels stabilize
- Produces finer-grained topic clusters
