# REST API Reference

Base URL: `https://api.trynebula.ai`

All requests require authentication via `Authorization: Bearer YOUR_API_KEY` or `X-API-Key: YOUR_API_KEY` header.

## Collections

### Create Collection
```
POST /v1/collections
```
```json
{"name": "research", "description": "Research papers", "metadata": {"category": "academic"}}
```

### List Collections
```
GET /v1/collections?limit=50&name=research
```

### Get Collection
```
GET /v1/collections/{collection_id}
```

### Update Collection
```
POST /v1/collections/{collection_id}
```
```json
{"name": "New Name", "description": "New description"}
```

### Delete Collection
```
DELETE /v1/collections/{collection_id}
```

## Memories

### Store Memory (Document)
```
POST /v1/memories
```
```json
{
  "collection_id": "COLLECTION_ID",
  "engram_type": "document",
  "raw_text": "Content to store",
  "metadata": {"key": "value"}
}
```

### Store Memory (Conversation)
```
POST /v1/memories
```
```json
{
  "collection_id": "COLLECTION_ID",
  "engram_type": "conversation",
  "messages": [
    {"role": "assistant", "content": "Hello!"},
    {"role": "user", "content": "Hi there"}
  ]
}
```

### Append to Conversation
```
POST /v1/memories
```
```json
{
  "memory_id": "EXISTING_MEMORY_ID",
  "collection_id": "COLLECTION_ID",
  "engram_type": "conversation",
  "messages": [{"role": "user", "content": "New message"}]
}
```

### Store Memory (File Upload)
```
POST /v1/memories
```
```json
{
  "collection_id": "COLLECTION_ID",
  "content_parts": [{
    "type": "document",
    "data": "BASE64_ENCODED_DATA",
    "media_type": "application/pdf",
    "filename": "document.pdf"
  }],
  "metadata": {"title": "Document"}
}
```

Inline base64 limited to ~5MB. For larger files, use presigned upload flow.

### Get Memory
```
GET /v1/engrams/{memory_id}
```

### List Memories
```
GET /v1/engrams?collection_ids=COLLECTION_ID&limit=50
```

With metadata filters (URL-encoded):
```
GET /v1/engrams?metadata_filters={"metadata.status":{"$eq":"active"}}
```

### Update Memory
```
POST /v1/memories/{memory_id}
```
```json
{"metadata": {"status": "reviewed"}, "merge_metadata": true}
```

### Delete Memory
```
DELETE /v1/engrams/{memory_id}
```

### Delete Multiple Memories
```
POST /v1/memories/delete
```
```json
{"ids": ["MEMORY_ID_1", "MEMORY_ID_2"]}
```

## Chunks

### Delete Chunk
```
DELETE /v1/chunks/{chunk_id}
```

### Update Chunk
```
PATCH /v1/chunks/{chunk_id}
```
```json
{"content": "Updated content", "metadata": {"edited": true}}
```

## Search

### Search Memories
```
POST /v1/memories/search
```
```json
{
  "query": "search text",
  "collection_ids": ["COLLECTION_ID"],
  "limit": 10,
  "offset": 0,
  "effort": "medium",
  "filters": {
    "metadata.topic": {"$eq": "AI"},
    "metadata.priority": {"$gte": 7}
  },
  "search_settings": {
    "semantic_weight": 0.8,
    "fulltext_weight": 0.2,
    "include_metadata": true
  }
}
```

Response:
```json
[
  {
    "memory_id": "eng_abc123",
    "content": "Matching content...",
    "score": 0.95,
    "metadata": {"topic": "AI"},
    "collection_id": "COLLECTION_ID",
    "created_at": "2024-01-15T10:30:00Z"
  }
]
```

## Conversations

### Get Conversation Messages
```
GET /v1/conversations/{conversation_id}
```

### List Conversations
```
GET /v1/conversations?limit=50
```

## Connectors

### List Providers
```
GET /v1/connectors/providers
```

### Connect Provider
```
POST /v1/connectors/connect
```

### List Connections
```
GET /v1/connectors/connections
```

### Get Connection
```
GET /v1/connectors/connections/{connection_id}
```

### Trigger Sync
```
POST /v1/connectors/connections/{connection_id}/sync
```

### Disconnect
```
DELETE /v1/connectors/connections/{connection_id}
```
