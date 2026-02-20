# Search Specification

## Overview

Xplor search combines full-text ranking (BM25) with semantic similarity
(cosine on embeddings) using Reciprocal Rank Fusion.

When embeddings are unavailable, falls back to BM25 only.

---

## Scoring Formula

```
finalScore = 0.6 * BM25_score + 0.3 * cosine_similarity + 0.1 * degreeBoost
```

### BM25

Standard BM25 with parameters: `k1 = 1.2`, `b = 0.75`.

### Cosine Similarity

If embeddings are available (pre-computed or on-demand):
- Use a lightweight model (e.g., `all-MiniLM-L6-v2` via Transformers.js for browser,
  or OpenAI/Anthropic embeddings API for server)
- Compare query embedding against node embeddings
- Score: 0.0 to 1.0

### Degree Boost

Highly connected nodes get a small relevance boost:
```javascript
degreeBoost = Math.min(1.0, (inDegree + outDegree) / 20);
```

---

## Fields Indexed

| Node Kind | Indexed Fields | Weight |
|-----------|---------------|--------|
| All | `name` | 3.0 |
| All | `description` | 2.0 |
| All | `tags` (joined) | 1.5 |
| Skill | `content.sections[].heading` | 1.5 |
| Skill | `content.sections[].preview` | 1.0 |
| Code | `metadata.signature` | 2.0 |
| Code | `source.filePath` | 1.0 |
| Document | `description` | 2.0 |

**`content.full` is NOT indexed by default.** Full content is too noisy for search
ranking. If the user explicitly requests deep search, index full content with weight 0.5.

---

## Reciprocal Rank Fusion (RRF)

When both BM25 and semantic results are available, fuse them:

```javascript
function reciprocalRankFusion(bm25Results, semanticResults, k = 60) {
  const scores = {};

  bm25Results.forEach((id, rank) => {
    scores[id] = (scores[id] || 0) + 1 / (k + rank + 1);
  });

  semanticResults.forEach((id, rank) => {
    scores[id] = (scores[id] || 0) + 1 / (k + rank + 1);
  });

  return Object.entries(scores)
    .sort(([, a], [, b]) => b - a)
    .map(([id]) => id);
}
```

---

## Fallback Behavior

| Condition | Behavior |
|-----------|----------|
| Embeddings available | Full hybrid: BM25 + cosine + degree |
| Embeddings unavailable | BM25 + degree only (cosine term = 0) |
| Empty query | Return nodes sorted by degree (most connected first) |
| No results | Return empty array, no error |

---

## Defaults

| Parameter | Default | Range |
|-----------|---------|-------|
| `limit` | 10 | 1-100 |
| `minScore` | 0.01 | 0-1 (filter out noise) |
| `deepSearch` | false | If true, index full content |

---

## Search API

### MCP Tool: `search`

```json
{
  "query": "authentication middleware",
  "limit": 10,
  "deepSearch": false
}
```

### Web API: `POST /api/search`

```json
{
  "graphId": "my-project",
  "query": "authentication middleware",
  "limit": 10,
  "kind": "code",
  "domain": null
}
```

### Response

```json
{
  "results": [
    {
      "id": "code:src/middleware/auth.ts#authMiddleware",
      "name": "authMiddleware",
      "type": "function",
      "score": 0.87,
      "description": "Express middleware for JWT token validation",
      "source": { "filePath": "src/middleware/auth.ts", "startLine": 8 }
    }
  ],
  "total": 5,
  "method": "hybrid"
}
```
