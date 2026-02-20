# API Route Contracts

All routes use Next.js App Router API routes (`app/api/*/route.js`).

---

## POST /api/extract

Extracts entities and connections from a text chunk using Claude.

### Request
```json
{
  "text": "chunk of PDF text (up to ~14,000 chars)",
  "fileName": "document.pdf",
  "chunkIndex": 0,
  "totalChunks": 3
}
```

### Response
```json
{
  "entities": [
    {
      "id": "person-john-smith",
      "name": "John Smith",
      "type": "person",
      "description": "CEO of Acme Corp mentioned in Q3 report"
    }
  ],
  "connections": [
    {
      "source": "person-john-smith",
      "target": "org-acme-corp",
      "label": "CEO of"
    }
  ]
}
```

### Implementation Notes
- Uses `claude-sonnet-4-20250514` for cost efficiency
- System prompt enforces JSON-only output
- max_tokens: 4096
- Parse with try/catch; return empty arrays on failure

---

## POST /api/share

Creates a public shared copy of a project.

### Request
```json
{
  "name": "My Analysis",
  "entities": [...],
  "connections": [...],
  "entityCount": 25,
  "connectionCount": 40,
  "documentCount": 3,
  "sharedBy": "User Display Name",
  "ownerUid": "firebase-uid"
}
```

### Response
```json
{
  "shareId": "abc12345",
  "url": "https://file-xplor.vercel.app/shared/abc12345"
}
```

### Implementation Notes
- Generates 8-char random alphanumeric ID
- Stores in `shared_projects/{shareId}` collection
- Sets `views: 0`, `sharedAt: serverTimestamp()`

---

## GET /api/share?id={shareId}

Retrieves a shared project and increments view count.

### Response
```json
{
  "name": "My Analysis",
  "entities": [...],
  "connections": [...],
  "sharedBy": "User Name",
  "sharedAt": "2025-02-15T...",
  "views": 142
}
```

---

## POST /api/chat

AI-powered Q&A about document contents. Pro feature.

### Request
```json
{
  "question": "What are the key financial entities?",
  "entities": [...],
  "connections": [...],
  "projectName": "Q3 Report Analysis"
}
```

### Response
```json
{
  "answer": "Based on the extracted data, the key financial entities are..."
}
```

### Implementation Notes
- Uses `claude-sonnet-4-20250514`
- System prompt includes entity/connection summary as context
- max_tokens: 1024
- Should check Pro status server-side (verify Firebase subscription)

---

## GET /api/trending

Returns top shared projects by view count.

### Query Params
- `limit` (optional, default 10)
- `period` (optional: "24h" | "7d" | "30d", default "24h")

### Response
```json
{
  "projects": [
    {
      "shareId": "abc12345",
      "name": "Contract Analysis",
      "entityCount": 45,
      "connectionCount": 78,
      "views": 342,
      "sharedBy": "Jane D.",
      "sharedAt": "2025-02-14T..."
    }
  ]
}
```

### Implementation Notes
- Queries `shared_projects` ordered by `views` descending
- Filter by `sharedAt` for time period
- Requires Firestore composite index on `(sharedAt, views)`

---

## POST /api/create-checkout

Creates a Stripe Checkout session for Pro subscription.

### Request
```json
{
  "uid": "firebase-uid",
  "email": "user@example.com"
}
```

### Response
```json
{
  "url": "https://checkout.stripe.com/c/pay_..."
}
```

---

## POST /api/webhooks/stripe

Handles Stripe webhook events.

### Events Handled
- `checkout.session.completed` — Mark user as Pro
- `customer.subscription.updated` — Update status
- `customer.subscription.deleted` — Remove Pro status

### Implementation Notes
- Verify webhook signature with `stripe.webhooks.constructEvent`
- Update `users/{uid}/subscription` in Firestore
- Raw body required (disable Next.js body parsing)
