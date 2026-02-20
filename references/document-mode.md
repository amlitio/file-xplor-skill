# Document Mode — Reference

## Overview

Document Mode is the user-facing web product. Users upload PDFs, AI extracts entities
and relationships, and the results render as an interactive knowledge graph.

## Files to Build (in order)

1. **`lib/firebase.js`** — Firebase init + Firestore helpers
   - `initializeApp`, `getAuth`, `getFirestore`
   - Functions: `saveProject`, `getProjects`, `deleteProject`, `logOut`
   - Google sign-in + email/password auth

2. **`lib/AuthContext.js`** — React context wrapping Firebase auth state
   - `useAuth()` hook returning `{ user, loading }`

3. **`app/layout.js`** — Root layout with AuthProvider, dark theme globals, Google Fonts

4. **`components/AuthScreen.js`** — Login/signup with Google + email

5. **`components/ProjectsDashboard.js`** — Grid of saved projects with delete

6. **`components/UploadScreen.js`** — PDF drop zone → client-side text extraction
   - Uses `pdfjs-dist` for client-side PDF text extraction
   - Chunks text at ~14,000 chars, sends up to 6 chunks to `/api/extract`
   - Deduplicates entities across chunks and files
   - Progress bar during extraction

7. **`app/api/extract/route.js`** — POST endpoint calling Claude for entity extraction

8. **`components/Explorer.js`** — Force-directed graph + sidebar + list view
   - See `references/explorer-architecture.md`

9. **`app/page.js`** — Screen router: landing → auth → dashboard → upload → explorer

10. **`components/ShareButton.js`** — Public share links with social distribution

11. **`app/shared/[id]/page.js`** — Public viewer (no auth, view counter, CTA)

12. **`app/api/share/route.js`** — Share CRUD

13. **`components/NetworkChat.js`** — AI Q&A floating panel (Pro feature)

14. **`app/api/chat/route.js`** — Chat endpoint

15. **`app/api/trending/route.js`** — Trending projects by views

16. **Stripe integration** — checkout, webhooks, Pro gating

## AI Entity Extraction Prompt

```
You are an entity extraction engine. Analyze the text and return ONLY valid JSON.
Extract entities (people, organizations, locations, dates, monetary amounts, concepts,
documents, events) and connections between them.

Return format:
{
  "entities": [
    { "id": "unique-id", "name": "Entity Name", "type": "person|organization|location|date|money|concept|document|event", "description": "Brief description" }
  ],
  "connections": [
    { "source": "entity-id-1", "target": "entity-id-2", "label": "relationship description" }
  ]
}

Be thorough. Extract 10-30 entities per chunk. Include cross-references.
```

Use `claude-sonnet-4-20250514` for cost efficiency. max_tokens: 4096.

## Firestore Schema

```
users/{uid}/projects/{projectId}:
  name, entities[], connections[], documents[],
  documentNames[], entityCount, connectionCount,
  documentCount, createdAt, updatedAt

shared_projects/{shareId}:
  name, entities[], connections[], entityCount,
  connectionCount, documentCount, sharedBy,
  sharedAt, views, ownerUid

users/{uid}/subscription:
  status, stripeCustomerId, stripePriceId, currentPeriodEnd
```

## Deployment

1. Push to GitHub
2. Connect to Vercel
3. Set environment variables
4. Enable Firebase Auth providers
5. Create Firestore indexes
6. Add Stripe webhook endpoint
7. Test full flow: upload → extract → graph → save → share → chat
