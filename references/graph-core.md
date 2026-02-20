# Graph Core — Canonical Data Model

## Purpose

Every mode (Document, Code, Skill Graph) produces nodes and edges. This file defines
the **single canonical schema** that all modes must conform to. This prevents integration
confusion, enables multi-domain fusion, and ensures every interface (Web, CLI, MCP)
speaks the same language.

**Rule: If it's a node or edge in Xplor, it conforms to this schema. No exceptions.**

---

## Graph Kinds

```typescript
type GraphKind = "document" | "code" | "skill" | "fused";
```

---

## GraphNode (Canonical)

```typescript
interface GraphNode {
  // --- Required ---
  id: string;                     // Globally unique, namespaced (see ID Rules below)
  kind: GraphKind;                // Which pipeline produced this node
  type: string;                   // Subtype within the kind (see Type Registry)
  name: string;                   // Display name

  // --- Recommended ---
  description?: string;           // Short summary (used in Level 1 scan)
  domain?: string;                // Grouping domain (therapy, trading, repo-name, etc.)
  tags?: string[];                // Freeform tags for filtering

  // --- Provenance ---
  source?: {
    filePath?: string;            // Relative path in repo/skill graph
    repo?: string;                // GitHub URL or local path
    documentId?: string;          // Firestore doc ID (for Document Mode)
    startLine?: number;           // Code entities: line range
    endLine?: number;
    updatedAt?: string;           // ISO 8601
  };

  // --- Progressive Disclosure Content ---
  content?: {
    full?: string;                // Level 4: complete content (markdown, code snippet)
    sections?: SectionPreview[];  // Level 3: headers + first paragraph
  };

  // --- Computed Metadata ---
  metadata?: {
    wordCount?: number;
    inDegree?: number;            // Computed after graph assembly
    outDegree?: number;
    signature?: string;           // Code: function signature
    language?: string;            // Code: programming language
    aliases?: string[];           // Skill: alternative names
  };
}

interface SectionPreview {
  heading: string;
  preview: string;                // First paragraph or ~100 words
  level: number;                  // Heading level (1-6)
}
```

---

## GraphEdge (Canonical)

```typescript
interface GraphEdge {
  // --- Required ---
  id: string;                     // Unique edge ID
  kind: GraphKind;                // Which pipeline produced this edge
  type: string;                   // Relationship type (see Edge Type Registry)
  source: string;                 // Source node ID
  target: string;                 // Target node ID

  // --- Recommended ---
  label?: string;                 // Human-readable label
  context?: string;               // Sentence containing the link/call (for ranking)
  weight?: number;                // 0-1, for weighted traversal
  domain?: string;                // For fusion: which domain this edge belongs to
}
```

---

## Node ID Rules

### Namespacing

IDs **must** be prefixed by kind to avoid collision across modes:

| Kind | Prefix | Example |
|------|--------|---------|
| Skill | `skill:` | `skill:cognitive-reframing` |
| Code | `code:` | `code:src/auth/validate.ts#validateToken` |
| Document | `doc:` | `doc:invoice-123#person-john-smith` |
| Fused | (original prefix preserved) | `skill:cognitive-reframing` stays as-is |

### Normalization Rules

1. Lowercase everything
2. Spaces → hyphens (`-`)
3. Strip `.md` extension
4. Slugify special characters
5. File-scoped entities use `#` separator: `code:src/auth.ts#login`

### Collision Strategy

During fusion, if two nodes from different graphs share the same name:
- Preserve both with their original namespaced IDs
- Create a `CROSS_DOMAIN` edge between them
- Never merge or overwrite

---

## Type Registry

### Document Mode Types

| type | Color | Description |
|------|-------|-------------|
| `person` | #FF6B6B | Named individual |
| `organization` | #4ECDC4 | Company, agency, institution |
| `location` | #45B7D1 | Place, address, region |
| `date` | #F7DC6F | Date or time reference |
| `money` | #BB8FCE | Currency, financial amount |
| `concept` | #82E0AA | Abstract idea, theme |
| `document` | #F0B27A | Referenced document |
| `event` | #AED6F1 | Named event, meeting, incident |

### Code Mode Types

| type | Color | Description |
|------|-------|-------------|
| `function` | #61AFEF | Function or method |
| `class` | #C678DD | Class or interface |
| `variable` | #E5C07B | Constant or variable |
| `import` | #56B6C2 | Import statement |
| `module` | #98C379 | Module or package |
| `file` | #ABB2BF | Source file |

### Skill Graph Types

| type | Color | Description |
|------|-------|-------------|
| `skill` | #FF9F43 | General knowledge node |
| `moc` | #EE5A24 | Map of Content (navigation hub) |
| `claim` | #A3CB38 | Methodology claim or assertion |
| `technique` | #FDA7DF | Actionable technique |
| `framework` | #9AECDB | Conceptual framework |
| `exploration` | #7158e2 | Open question or research area |

---

## Edge Type Registry

| type | Used In | Style | Description |
|------|---------|-------|-------------|
| `REFERENCES` | Skill | solid gray | Default wikilink |
| `CLUSTERS` | Skill | solid orange | MOC → child node |
| `EXTENDS` | Skill, Code | solid purple | Inheritance / builds-on |
| `CONTRADICTS` | Skill | dashed red | Opposing claim |
| `CALLS` | Code | dashed blue | Function invocation |
| `IMPORTS` | Code | solid teal | Module dependency |
| `DEFINES` | Code | dotted gray | File contains entity |
| `USES` | Code | dotted yellow | Variable reference |
| `EXPORTS` | Code | solid green | Public API surface |
| `RELATED_TO` | Document | solid gray | AI-extracted relationship |
| `CROSS_DOMAIN` | Fused | thick dashed gold | Cross-graph link |

---

## Graph Envelope

The top-level container for any graph:

```typescript
interface Graph {
  id: string;                     // Graph identifier
  kind: GraphKind;
  schemaVersion: "1.0.0";
  xplorVersion: "1.0.0";

  nodes: GraphNode[];
  edges: GraphEdge[];

  // Runtime indexes (built on load, not persisted)
  index?: {
    byId: Map<string, GraphNode>;
    adjacency: Map<string, string[]>;       // nodeId → outgoing neighbor IDs
    reverseAdjacency: Map<string, string[]>; // nodeId → incoming neighbor IDs
  };

  metrics: GraphMetrics;
}

interface GraphMetrics {
  nodeCount: number;
  edgeCount: number;
  density: number;                // edges / (nodes * (nodes - 1))
  domains: string[];
  typeBreakdown: Record<string, number>;
  edgeTypeBreakdown: Record<string, number>;
  avgDegree: number;
  maxInDegree: { nodeId: string; value: number };
  maxOutDegree: { nodeId: string; value: number };
  orphanCount: number;
  clusterCount: number;           // Connected components
  mocCount?: number;              // Skill graphs only
}
```

---

## Serialization (JSON Persistence)

### Storage Location

```
~/.xplor/
├── config.json
└── graphs/
    ├── <graphId>.json            # Full graph
    └── <graphId>.meta.json       # Metadata only (for listing)
```

### `<graphId>.json`

```json
{
  "schemaVersion": "1.0.0",
  "xplorVersion": "1.0.0",
  "id": "therapy-cbt",
  "kind": "skill",
  "nodes": [...],
  "edges": [...],
  "metrics": {...}
}
```

### `<graphId>.meta.json`

```json
{
  "id": "therapy-cbt",
  "kind": "skill",
  "sourcePath": "./therapy-cbt",
  "createdAt": "2025-02-19T...",
  "hash": "sha256-of-all-input-files",
  "nodeCount": 24,
  "edgeCount": 67,
  "domains": ["therapy"]
}
```

### Versioning Rules

- `schemaVersion` tracks the data model (this spec)
- `xplorVersion` tracks the tool version
- Minor version bumps must be backward-compatible
- Major version bumps may require migration

### Incremental Re-index

- Compare `hash` of input files against stored `meta.json`
- If hash matches: load from cache
- If hash differs: re-run pipeline, overwrite
- CLI flag: `xplor index --force` to skip hash check
