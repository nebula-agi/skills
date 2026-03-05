# JavaScript / TypeScript SDK Reference

Package: `@nebula-ai/sdk` | Node.js >= 18 | Full TypeScript types included

## Installation

```bash
npm install @nebula-ai/sdk
```

## Client Initialization

```typescript
import { Nebula } from '@nebula-ai/sdk';
// or: const { Nebula } = require('@nebula-ai/sdk');

// Auto-reads NEBULA_API_KEY env var
const nebula = new Nebula();

// Explicit config
const nebula = new Nebula({
  apiKey: 'your-api-key',
  baseUrl: 'https://api.trynebula.ai',  // default
  timeout: 30000  // milliseconds, default
});
```

All methods are async and return Promises.

## Collections

```typescript
// Create
const collection = await nebula.createCollection({
  name: 'research',
  description: 'Research papers',
  metadata: { category: 'academic' }
});
// collection.id -> UUID string

// List
const collections = await nebula.listCollections({ limit: 50 });
const filtered = await nebula.listCollections({ name: 'research' });

// Get by ID
const collection = await nebula.getCollection(collectionId);

// Get by name
const collection = await nebula.getCollectionByName('research');

// Update
await nebula.updateCollection({
  collectionId: collection.id,
  name: 'New Name',
  description: 'New description'
});

// Delete (permanently removes all memories within)
await nebula.deleteCollection(collectionId);
```

## Storing Memories

### Single Memory

```typescript
const memoryId = await nebula.storeMemory({
  collection_id: collection.id,
  content: 'Some text content',
  metadata: { key: 'value' },
  authority: 0.8  // optional, 0-1, default 0.5
});
```

### Batch Storage

```typescript
const memoryIds = await nebula.storeMemories([
  { collection_id: cid, content: 'First', metadata: { n: 1 } },
  { collection_id: cid, content: 'Second', metadata: { n: 2 } }
]);
```

### Conversation Messages

```typescript
// Create conversation
const convId = await nebula.storeMemory({
  collection_id: cid,
  content: 'Hello!',
  role: 'assistant'
});

// Append single message
await nebula.storeMemory({
  memory_id: convId,
  collection_id: cid,
  content: 'I need help',
  role: 'user'
});

// Append multiple messages
await nebula.storeMemory({
  memory_id: convId,
  collection_id: cid,
  content: [
    { content: 'What issue?', role: 'assistant' },
    { content: 'Login broken', role: 'user' }
  ]
});
```

### Document Upload

```typescript
import Nebula, { Memory } from '@nebula-ai/sdk';

// Text (auto-chunked)
const docId = await nebula.storeMemory({
  collection_id: cid,
  content: 'Full document content...',
  metadata: { title: 'Paper' }
});

// Pre-chunked
const chunkedId = await nebula.storeMemory({
  collection_id: cid,
  content: ['Chapter 1...', 'Chapter 2...'],
  metadata: { title: 'Book' }
});

// File upload (Node.js only)
const fileId = await nebula.storeMemory(
  await Memory.fromFile('document.pdf', cid, { title: 'Research' })
);
```

Supported file types: PDF, DOC, DOCX, TXT, JPEG, PNG, GIF, WebP, MP3, WAV, M4A, OGG, FLAC. Files > 5MB use presigned S3 upload (max 100MB).

## Retrieving Memories

```typescript
// Get by ID (includes chunks)
const memory = await nebula.getMemory(memoryId);
// memory.chunks -> array of { id, content, metadata }

// List memories
const memories = await nebula.listMemories({
  collection_ids: ['research'],
  limit: 50,
  metadata_filters: { 'metadata.status': { $eq: 'active' } }
});

// Get conversation messages
const messages = await nebula.getConversationMessages(convId);

// List conversations
const conversations = await nebula.listConversations({ limit: 50 });
```

## Updating

```typescript
// Update memory metadata (does NOT update content)
await nebula.updateMemory({
  memoryId: mid,
  metadata: { status: 'reviewed' },
  mergeMetadata: true
});

// Update specific chunk
await nebula.updateChunk(chunkId, 'Updated content', { edited: true });
```

## Deleting

```typescript
// Single memory
await nebula.delete(memoryId);

// Multiple memories
await nebula.delete([mid1, mid2, mid3]);

// Single chunk
await nebula.deleteChunk(chunkId);
```

## Search

```typescript
const results = await nebula.search({
  query: 'machine learning',
  collection_ids: ['research'],       // optional, omit for all
  limit: 10,                          // optional
  offset: 0,                          // optional, pagination
  filters: { 'metadata.topic': 'AI' },// optional
  effort: 'medium',                   // optional: auto/low/medium/high
  searchSettings: {                   // optional
    semanticWeight: 0.8,              // default
    fulltextWeight: 0.2,              // default
    includeMetadata: true             // return metadata in results
  }
});

for (const r of results) {
  console.log(r.score);          // 0-1 relevance
  console.log(r.content);        // chunk content
  console.log(r.metadata);       // metadata object
  console.log(r.memory_id);      // parent memory ID
  console.log(r.collection_id);  // collection ID
  console.log(r.created_at);     // timestamp
}
```

## Error Handling

The SDK provides typed exception classes:

```typescript
import {
  NebulaException,                // base
  NebulaClientException,          // config/setup errors
  NebulaAuthenticationException,  // 401
  NebulaRateLimitException,       // 429
  NebulaValidationException,      // 400
  NebulaNotFoundException,        // 404
  NebulaCollectionNotFoundException
} from '@nebula-ai/sdk';

try {
  const results = await nebula.search({ query: 'test' });
} catch (e) {
  if (e instanceof NebulaAuthenticationException) {
    console.error('Invalid API key');
  } else if (e instanceof NebulaRateLimitException) {
    console.error('Rate limited, retry later');
  } else if (e instanceof NebulaException) {
    console.error(`Nebula error: ${e.message}`);
  }
}
```

## Memory Factory Helpers

```typescript
import { Memory } from '@nebula-ai/sdk';

// Create from file path (Node.js only)
const mem = await Memory.fromFile('doc.pdf', collectionId, { title: 'Doc' });

// Create file memory object
const mem = Memory.File(buffer, 'doc.pdf', collectionId, { title: 'Doc' });
```

## Utilities

```typescript
// Health check
await nebula.healthCheck();  // throws on failure

// Get presigned upload URL
const urlInfo = await nebula.getUploadUrl();
```
