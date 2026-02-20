# Code Mode — Ingestion Pipeline Reference

## Overview

The code ingestion pipeline transforms a repository into a knowledge graph:
1. Discover source files
2. Parse each into an AST with Tree-sitter
3. Extract code entities (functions, classes, imports)
4. Build relationship edges (CALLS, IMPORTS, EXTENDS, DEFINES)

## File Discovery

```javascript
import { glob } from "glob";

const LANGUAGE_MAP = {
  ".js": "javascript", ".jsx": "javascript",
  ".ts": "typescript", ".tsx": "typescript",
  ".py": "python",
};

const IGNORE = ["**/node_modules/**", "**/dist/**", "**/.git/**", "**/*.min.js"];

async function discoverFiles(repoPath, languages = null) {
  const extensions = languages
    ? languages.flatMap((l) => Object.entries(LANGUAGE_MAP).filter(([, v]) => v === l).map(([k]) => k))
    : Object.keys(LANGUAGE_MAP);
  return glob(`**/*{${extensions.join(",")}}`, { cwd: repoPath, ignore: IGNORE, absolute: true });
}
```

## AST Parsing with Tree-sitter

```javascript
import Parser from "web-tree-sitter";

async function parseFile(filePath, language, source) {
  await Parser.init();
  const parser = new Parser();
  const lang = await Parser.Language.load(`grammars/tree-sitter-${language}.wasm`);
  parser.setLanguage(lang);
  return parser.parse(source);
}
```

## Entity Extraction

Walk the AST extracting:

| AST Node | Entity Type | Fields |
|----------|------------|--------|
| `function_declaration` | function | name, params, return type, line range |
| `arrow_function` (assigned) | function | name from variable, params |
| `method_definition` | function | name, class, params |
| `class_declaration` | class | name, extends, methods |
| `import_statement` | import | source, specifiers |
| `variable_declarator` | variable | name, init type |

### Code Entity Schema

```typescript
interface CodeEntity {
  id: string;           // "func-src/auth/validate.ts-validateToken"
  name: string;         // "validateToken"
  type: "function" | "class" | "variable" | "import" | "module" | "file";
  filePath: string;     // "src/auth/validate.ts"
  language: string;     // "typescript"
  startLine: number;
  endLine: number;
  signature?: string;   // "validateToken(token: string): boolean"
}
```

## Relationship Extraction

| Relationship | How Detected | Meaning |
|-------------|-------------|---------|
| CALLS | `call_expression` where callee matches known function | Function invokes function |
| IMPORTS | `import_statement` resolved to target file | File depends on file |
| EXTENDS | `class_declaration` with superclass | Class inherits from class |
| DEFINES | Function/class found inside file | File contains entity |
| USES | Variable reference inside function body | Function reads variable |
| EXPORTS | `export_statement` | File exposes entity |

## Full Pipeline

```javascript
async function runIngestionPipeline(repoPath, options = {}) {
  const files = await discoverFiles(repoPath, options.languages);
  await Parser.init();
  
  const allEntities = [];
  const allEdges = [];
  
  for (const file of files) {
    const source = await fs.readFile(file, "utf-8");
    const tree = await parseFile(file, detectLanguage(file), source);
    
    // File entity
    allEntities.push({ id: `file-${file}`, name: file, type: "file", filePath: file });
    
    // Extract entities + DEFINES edges
    const entities = extractEntities(tree, file);
    allEntities.push(...entities);
    entities.forEach((e) => allEdges.push({ source: `file-${file}`, target: e.id, type: "DEFINES" }));
  }
  
  // Cross-file: CALLS, IMPORTS, EXTENDS
  const entityIndex = new Map(allEntities.map((e) => [e.name, e.id]));
  // ... resolve cross-references
  
  return { entities: allEntities, connections: allEdges };
}
```

## Web Mode (Browser)

Same pipeline runs in a Web Worker with WASM Tree-sitter. Accepts GitHub URL
(clone via isomorphic-git), ZIP upload, or pasted code.
