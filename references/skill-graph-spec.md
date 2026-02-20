# Skill Graph Mode — Implementation Spec

## Overview

A skill graph is a network of markdown files connected with `[[wikilinks]]`.
Each file = one concept. Links = traversable relationships. AI agents navigate
the graph progressively, loading only what's needed.

All nodes and edges conform to the canonical schema in `references/graph-core.md`.

---

## Ingestion Pipeline

### Step 1 — File Discovery

Walk directory for `.md` files. Ignore: `.git`, `node_modules`, `dist`, `.next`, `build`.

### Step 2 — Frontmatter Extraction

Parse YAML with `gray-matter`. Required: `description`. Optional: `name`, `type`,
`domain`, `tags`, `aliases`, `extends`, `contradicts`.

### Step 3 — Wikilink Extraction

Regex (supports target, anchor, alias):
```
\[\[([^\]|#]+)(?:#([^\]|]+))?(?:\|([^\]]+))?\]\]
```

**Sentence-level context** (not line-level — this matters for traversal ranking):

```javascript
function extractWikilinks(content) {
  // Split into sentences first
  const sentences = content.match(/[^.!?\n]+[.!?\n]+/g) || [content];
  const links = [];

  for (const sentence of sentences) {
    let match;
    const re = /\[\[([^\]|#]+)(?:#([^\]|]+))?(?:\|([^\]]+))?\]\]/g;
    while ((match = re.exec(sentence)) !== null) {
      links.push({
        target: slugify(match[1].trim()),
        anchor: match[2] || null,
        alias: match[3] || null,
        contextSentence: sentence.trim(),  // sentence, NOT full line
      });
    }
  }

  return links;
}

function slugify(str) {
  return str.toLowerCase().replace(/\.md$/, "").replace(/\s+/g, "-").replace(/[^a-z0-9-]/g, "");
}
```

### Step 4 — Node Construction

Each file becomes a `GraphNode` per `graph-core.md`:

```javascript
{
  id: `skill:${slug}`,
  kind: "skill",
  type: frontmatter.type || "skill",
  name: frontmatter.name || titleFromFilename,
  description: frontmatter.description,
  domain: frontmatter.domain || "general",
  tags: frontmatter.tags || [],
  source: { filePath: relativePath, updatedAt: stat.mtime },
  content: {
    full: markdownBody,
    sections: extractSections(markdownBody),
  },
  metadata: {
    wordCount: markdownBody.split(/\s+/).length,
    aliases: frontmatter.aliases || [],
  },
}
```

### Step 5 — Edge Construction

| Condition | Edge Type |
|-----------|----------|
| Default wikilink | `REFERENCES` |
| Source node type is `moc` | `CLUSTERS` |
| Declared in frontmatter `extends: [...]` | `EXTENDS` |
| Declared in frontmatter `contradicts: [...]` | `CONTRADICTS` |

Each edge includes `contextSentence` from Step 3.

### Step 6 — Metrics & Validation

Compute `GraphMetrics` per `graph-core.md`. Run quality validation per
`references/skill-graph-quality.md`. Use `references/progressive-disclosure.md`
to pre-compute section previews.

---

## Maps of Content (MOCs)

MOCs are navigation entry points. They cluster related nodes and direct agent attention.

An MOC is identified by: `type: moc` in frontmatter.

Rules:
- MOCs should link to 5-15 children (too few = useless, too many = noisy)
- MOCs should have a narrative, not just a link list
- Wikilinks in MOCs generate `CLUSTERS` edges (stronger than `REFERENCES`)
- A well-structured graph has 1 MOC per 5-10 skill nodes

### Example MOC

```markdown
---
name: cbt-techniques
description: >
  Map of Content for CBT techniques. Entry point for agents working
  on cognitive behavioral therapy tasks.
type: moc
domain: therapy
---

# CBT Techniques

Core techniques for cognitive behavioral therapy sessions.

## Cognitive Techniques
- [[cognitive-reframing]] — challenging distorted thoughts
- [[thought-records]] — structured thought analysis worksheets
- [[cognitive-distortions]] — taxonomy of common patterns
- [[socratic-questioning]] — guided discovery through questions

## Behavioral Techniques
- [[behavioral-activation]] — scheduling positive activities
- [[exposure-hierarchy]] — graduated exposure to fears
```

---

## Traversal

See `references/progressive-disclosure.md` for the 5-level enforcement rules.
See `references/agent-intelligence.md` for attention scoring and path ranking.
See `references/skill-graph-api.md` for the web API contracts.

The traversal pattern:
```
Agent receives task
  → Index (Level 0): what MOCs exist?
  → Scan (Level 1): which nodes match the task?
  → Links (Level 2): what's connected to the matches?
  → Sections (Level 3): confirm relevance with previews
  → Full (Level 4): load 3-8 most relevant nodes
  → Assemble context pack within token budget
```

---

## CLI Commands

See `references/cli-spec.md` for full details.

```bash
xplor skill init <n>          # Scaffold new skill graph
xplor skill index <path>         # Build graph + write JSON
xplor skill validate <path>      # Quality score + issue report
xplor skill stats <path>         # Topology metrics
xplor skill viz <path> --open    # Visual graph in browser
```

---

## Example Skill Graphs

### Therapy (CBT)
```
therapy-cbt/
├── index.md (moc)
├── techniques/
│   ├── cognitive-reframing.md → [[thought-records]], [[cognitive-distortions]]
│   ├── thought-records.md → [[socratic-questioning]]
│   └── exposure-hierarchy.md → [[behavioral-activation]]
├── claims/
│   └── validation-first.md → [[grounding-techniques]]
└── frameworks/
    └── case-formulation.md → integrates all techniques
```

### Trading
```
trading/
├── index.md (moc)
├── mocs/risk-management.md, market-structure.md, psychology.md, execution.md
├── techniques/position-sizing.md, stop-loss-strategies.md, scaling-in-out.md
├── claims/risk-first-not-reward-first.md, process-over-outcome.md
└── frameworks/expected-value.md, kelly-criterion.md, regime-detection.md
```

### Company Knowledge
```
acme-corp/
├── index.md (moc)
├── mocs/org-structure.md, product-knowledge.md, processes.md, culture.md
├── products/widget-pro.md, widget-enterprise.md, pricing-model.md
├── processes/incident-response.md, deployment-pipeline.md, code-review.md
└── culture/decision-making.md, communication-norms.md, values.md
```
