# Explorer Component Architecture — Mode-Aware

The Explorer renders all three graph types. It adapts its UI based on
`graph.kind`: document, code, or skill.

---

## Props

```typescript
interface ExplorerProps {
  data: {
    id?: string;
    name?: string;
    kind: "document" | "code" | "skill" | "fused";
    nodes: GraphNode[];       // Canonical schema from graph-core.md
    edges: GraphEdge[];
    metrics?: GraphMetrics;
    documents?: DocMeta[];    // Document mode only
  };
  onBack: () => void;
}
```

---

## Layout

```
Explorer
├── Top Bar
│   ├── Back button
│   ├── Title + mode badge + stats
│   ├── Mode indicator (📄 Document | 💻 Code | 🧠 Skill Graph | 🔀 Fused)
│   ├── View toggle (Graph / List / MOC Index)
│   ├── Domain filter (fused graphs: toggle domains on/off)
│   ├── ShareButton
│   └── Save button
│
├── Main Content (flex row)
│   ├── Graph View (SVG canvas)
│   │   ├── Nodes (shape varies by mode — see Node Rendering)
│   │   ├── Edges (style varies by edge type — see Edge Rendering)
│   │   ├── Domain color regions (fused mode: cluster by domain)
│   │   ├── Telemetry heatmap overlay (skill mode: glow = loaded)
│   │   ├── Zoom/pan controls
│   │   └── Legend (entity types + edge types)
│   │
│   ├── List View (searchable grid of entity cards)
│   │
│   ├── MOC Index View (skill mode only)
│   │   ├── MOC cards with child counts
│   │   ├── Click MOC → filter graph to its cluster
│   │   └── Breadcrumb trail: Graph → MOC → Node
│   │
│   └── Detail Sidebar (right, 360px)
│       ├── Tab: Info (name, type, description, source)
│       ├── Tab: Links (outgoing + incoming connections)
│       ├── Tab: Sections (Level 3 previews with "Load Full" button)
│       ├── Tab: Full (Level 4 complete content — skill/code only)
│       └── Tab: Telemetry (was this node loaded? score? reason?)
│
└── Bottom Panels
    ├── NetworkChat (floating, bottom-right — all modes)
    └── Traversal Breadcrumbs (skill mode — shows navigation path)
```

---

## Node Rendering (by kind)

### Document Mode
- **Shape:** Circle
- **Size:** `radius = max(6, min(16, 6 + connectionCount * 1.5))`
- **Color:** Entity type color from graph-core.md
- **Label:** Entity name (truncated to 16 chars)

### Code Mode
- **Shape:** Rounded rectangle (functions/classes), circle (files/imports)
- **Size:** Width proportional to name length, height fixed
- **Color:** Code type color from graph-core.md
- **Label:** Function/class name + file indicator
- **Extra:** Dashed border for imported/external entities

### Skill Graph Mode
- **Shape:** Rectangular card (skill nodes), larger bordered rectangle (MOCs)
- **Size:** Cards are ~120x50px, MOCs are ~160x60px
- **Color:** Skill type color from graph-core.md
- **Label:** Node name, type badge below
- **Extra:** MOCs get a thicker border + subtle glow
- **Telemetry:** If telemetry data is available:
  - Loaded nodes: full opacity + bright glow
  - Visited but skipped: 40% opacity
  - Not visited: 20% opacity

### Fused Mode
- **Shape:** Based on original kind (document=circle, code=rounded-rect, skill=card)
- **Extra:** Domain-colored background regions cluster nodes by domain
- **CROSS_DOMAIN edges:** Thick dashed gold lines between domains

---

## Edge Rendering (by type)

| Edge Type | Style | Color | Width |
|-----------|-------|-------|-------|
| REFERENCES | Solid | rgba(255,255,255,0.1) | 1 |
| CLUSTERS | Solid | #EE5A24 at 30% | 1.5 |
| EXTENDS | Solid | #C678DD at 40% | 1.5 |
| CONTRADICTS | Dashed | #FF6B6B at 40% | 1.5 |
| CALLS | Dashed | #61AFEF at 30% | 1 |
| IMPORTS | Solid | #56B6C2 at 30% | 1 |
| DEFINES | Dotted | rgba(255,255,255,0.06) | 0.8 |
| RELATED_TO | Solid | rgba(255,255,255,0.08) | 1 |
| CROSS_DOMAIN | Thick dashed | #F7DC6F at 50% | 2.5 |

Edge labels appear at the midpoint, 8px font, only on hover or when zoomed in.

---

## Sidebar Tabs

### Info Tab (all modes)
- Node name with colored type dot
- Type badge
- Domain badge (if present)
- Description
- Source info (file path, line range, document name)
- Tags

### Links Tab (all modes)
- Outgoing connections: list with type labels
- Incoming connections: list with type labels
- Each link is clickable → navigates to that node

### Sections Tab (skill + code modes)
- Renders `content.sections[]` from the node
- Each section: heading + preview text
- "Load Full" button at bottom → switches to Full tab

### Full Tab (skill + code modes)
- Complete `content.full` rendered as markdown (skill) or syntax-highlighted code (code)
- Only loads when explicitly requested (progressive disclosure enforcement)

### Telemetry Tab (skill mode, when telemetry available)
- **Score:** Attention score for this node in current traversal
- **Level loaded:** Which progressive disclosure level was applied
- **Reason:** Why this node was loaded/skipped
- **Connections loaded:** Which of this node's neighbors were also loaded
- **Traversal position:** Depth in the traversal tree

---

## MOC Index View (Skill Mode Only)

When `kind === "skill"`, a third view is available: MOC Index.

Renders all MOC nodes as large cards in a grid:

```
┌─────────────────────┐  ┌─────────────────────┐
│  🗺️ CBT Techniques  │  │  🗺️ Assessment      │
│  12 children         │  │  6 children          │
│  therapy domain      │  │  therapy domain      │
└─────────────────────┘  └─────────────────────┘
```

Click a MOC → Graph view filters to show only that MOC's cluster.
Breadcrumbs update: `All Nodes > CBT Techniques > [selected node]`

---

## Traversal Breadcrumbs (Skill Mode)

A horizontal breadcrumb bar below the top bar showing the agent's navigation path:

```
🧠 therapy-cbt > 🗺️ cbt-techniques > 📝 cognitive-distortions > 📝 cognitive-reframing
```

Click any breadcrumb → navigate back to that node in the graph.

---

## Telemetry Heatmap Overlay (Skill Mode)

When traversal telemetry data is available (from `/api/skillgraph/traverse`),
overlay it on the graph:

- **Loaded nodes (Level 3-4):** Full brightness, colored glow matching type
- **Visited nodes (Level 1-2):** 40% opacity, subtle border
- **Unvisited nodes:** 20% opacity, no border
- **Coverage meter:** Small progress bar in bottom-left showing `loadedNodes / totalNodes`

Toggle the overlay with a button in the toolbar.

---

## State

```typescript
const [mode, setMode] = useState<"graph" | "list" | "moc">("graph");
const [selectedNode, setSelectedNode] = useState<GraphNode | null>(null);
const [sidebarTab, setSidebarTab] = useState<"info" | "links" | "sections" | "full" | "telemetry">("info");
const [domainFilter, setDomainFilter] = useState<Set<string>>(new Set()); // fused mode
const [telemetryData, setTelemetryData] = useState(null);
const [breadcrumbs, setBreadcrumbs] = useState<string[]>([]);
const [searchQuery, setSearchQuery] = useState("");
const [transform, setTransform] = useState({ x: 0, y: 0, k: 1 });
```
