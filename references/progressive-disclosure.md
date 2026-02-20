# Progressive Disclosure — Enforcement Rules

## Purpose

Progressive disclosure prevents agents from loading entire knowledge graphs into context.
Instead, agents navigate: scan descriptions → follow links → read sections → load full
content for only the most relevant nodes.

**These rules are mandatory for every interface: Web API, CLI, MCP tools.**

---

## The Five Levels

| Level | Name | What's Returned | What's Forbidden |
|-------|------|----------------|------------------|
| 0 | **Index** | Node IDs, names, domains, types, MOC list | Any content, descriptions, links |
| 1 | **Scan** | Frontmatter only: id, name, type, domain, description, tags | Markdown body, sections, links, full content |
| 2 | **Links** | Link targets + context sentences + target descriptions | Section previews, full content |
| 3 | **Sections** | Headings + first paragraph per section (~100 words each) | Full content body |
| 4 | **Full** | Complete markdown/code content | Nothing forbidden |

---

## Enforcement Rules

### Rule 1: Level field is always present

Every node returned by any API must include a `level` field indicating which
disclosure level was applied.

```json
{ "id": "skill:cognitive-reframing", "level": 1, "description": "..." }
```

### Rule 2: Forbidden fields are NEVER included

If a node is returned at Level 1 (scan), the response must NOT contain:
- `content.full`
- `content.sections`
- `links` array

Violations are bugs, not features. Even if the data is available in memory,
it must be stripped before returning.

### Rule 3: Levels are monotonically upgradeable

An agent can request a higher level for any node it's already seen:
- See at Level 1 → request Level 3 → get sections
- See at Level 3 → request Level 4 → get full content
- NEVER downgrade: requesting Level 1 for a node already loaded at Level 4 still returns Level 4

### Rule 4: Token budget is a hard ceiling

When `tokenBudget` is specified:
- Estimate tokens for each node before including it
- Stop adding nodes when budget would be exceeded
- Prefer adding more nodes at lower levels over fewer at higher levels
- Report `tokensUsed` and `tokenBudget` in telemetry

Token estimation: `tokens ≈ wordCount * 1.3` (rough but sufficient for v1)

### Rule 5: Every operation emits telemetry

Every API call that touches nodes must return:

```json
{
  "telemetry": {
    "nodesVisited": 12,
    "nodesLoaded": 5,
    "nodesAtLevel": { "0": 0, "1": 4, "2": 0, "3": 2, "4": 3 },
    "tokensUsed": 4200,
    "tokenBudget": 6000,
    "coveragePercent": 20.8
  }
}
```

---

## Per-Interface Enforcement

### Web API

| Endpoint | Default Level | Upgradeable |
|----------|--------------|-------------|
| `POST /api/skillgraph/scan` | 1 | No (scan is always Level 1) |
| `POST /api/skillgraph/traverse` | Mixed (auto-selected per node) | No (server decides) |
| `POST /api/skillgraph/context` | Mixed (optimized for token budget) | No |
| `GET /api/skillgraph/stats` | 0 (metrics only, no nodes) | No |

### MCP Tools

| Tool | Default Level | Upgradeable |
|------|--------------|-------------|
| `scan` | 1 | No |
| `traverse` | Mixed (3-4 for top nodes, 1 for rest) | No |
| `context` | Mixed (auto) | No |
| `query` | 1 (results) + 3 (top result) | Yes, agent can call `context` for Level 4 |
| `search` | 1 | Yes |

### CLI

| Command | Default Level |
|---------|--------------|
| `xplor skill stats` | 0 |
| `xplor query` | 1 with Level 3 for top results |

---

## Caching Strategy

### Frontmatter Index (Level 0-1)

- Cached in memory on graph load
- Contains: all node IDs, names, types, domains, descriptions, tags
- Size: ~100 bytes per node → 10KB for 100-node graph
- Never evicted during session

### Section Previews (Level 3)

- Computed on first request, cached per node
- Contains: heading text + first ~100 words per section
- Size: ~500 bytes per node
- Evicted when graph is unloaded

### Full Content (Level 4)

- Loaded on demand only
- Never pre-cached for entire graph
- Cached per-node during session
- Evicted on session end

---

## Section Extraction Algorithm

For Level 3, extract section previews from markdown:

```javascript
function extractSections(markdown) {
  const sections = [];
  const lines = markdown.split("\n");
  let currentHeading = null;
  let currentContent = [];

  for (const line of lines) {
    const headingMatch = line.match(/^(#{1,6})\s+(.+)/);
    if (headingMatch) {
      // Save previous section
      if (currentHeading) {
        sections.push({
          heading: currentHeading.text,
          level: currentHeading.level,
          preview: currentContent.join(" ").slice(0, 400).trim(),
        });
      }
      currentHeading = {
        text: headingMatch[2],
        level: headingMatch[1].length,
      };
      currentContent = [];
    } else if (line.trim()) {
      currentContent.push(line.trim());
    }
  }

  // Don't forget last section
  if (currentHeading) {
    sections.push({
      heading: currentHeading.text,
      level: currentHeading.level,
      preview: currentContent.join(" ").slice(0, 400).trim(),
    });
  }

  return sections;
}
```
