# Security & Limits

## Web Ingestion Limits

| Limit | Value | Applies To |
|-------|-------|-----------|
| Max ZIP upload size | 10 MB | Skill Graph + Code Mode |
| Max file count per upload | 500 files | All modes |
| Max total markdown content | 5 MB | Skill Graph Mode |
| Max individual file size | 1 MB | All modes |
| Max PDF size | 25 MB | Document Mode |
| Max PDFs per upload | 10 files | Document Mode |

## Rate Limits

| Operation | Limit | Window |
|-----------|-------|--------|
| Ingestion (ZIP/GitHub/PDF) | 20 requests | per hour per user |
| AI extraction (Claude API) | 60 chunks | per hour per user |
| AI chat | 30 messages | per hour per user (Pro: 120) |
| Share creation | 10 shares | per hour per user |
| Scan/traverse/stats | 100 requests | per minute per user |

## File Type Allowlist

### Skill Graph Mode
Only `.md` files are parsed. All others are ignored silently.

### Code Mode
Only files matching the language map extensions are parsed:
`.js`, `.jsx`, `.ts`, `.tsx`, `.py`

All others (images, binaries, configs) are ignored.

### Document Mode
Only `.pdf` files accepted. Other formats return `400`.

## Sandboxing

### Browser (Web Workers)
- Code parsing (Tree-sitter WASM) runs in a dedicated Web Worker
- Skill graph parsing runs in a dedicated Web Worker
- Workers have no DOM access
- Workers are terminated after 60-second timeout

### Server (API Routes)
- ZIP extraction uses streaming decompression with size checks
- GitHub clone uses shallow clone (`--depth 1`) with timeout
- No shell execution from user input
- File paths are sanitized (no `..` traversal, no symlinks followed)

## Pro Feature Gating

| Feature | Free | Pro ($9.99/mo) |
|---------|------|----------------|
| PDF uploads | 3/day | Unlimited |
| AI chat | ❌ | ✅ |
| Code Mode | 1 repo/day | Unlimited |
| Skill Graph Mode | 1 graph/day | Unlimited |
| Shared project views | ✅ | ✅ |
| Traversal telemetry | Basic | Full with export |
| Multi-domain fusion | ❌ | ✅ |
| Priority API rate limits | Standard | 4x multiplier |

## Input Sanitization

- All user-provided strings (project names, queries) are escaped before
  rendering in HTML or using in Firestore queries
- File names are sanitized to alphanumeric + hyphens + dots
- Markdown content is rendered with DOMPurify when displayed in the web UI
- Wikilink targets are slugified before graph lookup (prevents injection)
