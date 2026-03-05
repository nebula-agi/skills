# Python SDK Reference

Package: `nebula-client` | Python >= 3.10 | Dependencies: httpx, tiktoken

## Installation

```bash
pip install nebula-client
```

## Client Initialization

```python
from nebula import Nebula

# Auto-reads NEBULA_API_KEY env var
nebula = Nebula()

# Explicit config
nebula = Nebula(
    api_key="your-api-key",
    base_url="https://api.trynebula.ai",  # default
    timeout=30.0  # seconds, default
)
```

## Async Client

```python
from nebula import AsyncNebula

# Context manager (recommended)
async with AsyncNebula() as nebula:
    results = await nebula.search(query="hello")

# Manual lifecycle
nebula = AsyncNebula()
results = await nebula.search(query="hello")
await nebula.aclose()
```

All methods below have async equivalents on `AsyncNebula` (prefix with `await`).

## Collections

```python
# Create
collection = nebula.create_collection(
    name="research",
    description="Research papers",
    metadata={"category": "academic"}
)
# collection.id -> UUID string

# List
collections = nebula.list_collections(limit=50)
collections = nebula.list_collections(name="research")  # filter by name

# Get by ID
collection = nebula.get_collection(collection_id)

# Get by name
collection = nebula.get_collection_by_name("research")

# Update
nebula.update_collection(
    collection_id=collection.id,
    name="New Name",
    description="New description"
)

# Delete (permanently removes all memories within)
nebula.delete_collection(collection_id)
```

## Storing Memories

### Single Memory

```python
memory_id = nebula.store_memory({
    "collection_id": collection.id,
    "content": "Some text content",
    "metadata": {"key": "value"},
    "authority": 0.8  # optional, 0-1, default 0.5
})
```

### Batch Storage

```python
memory_ids = nebula.store_memories([
    {"collection_id": cid, "content": "First", "metadata": {"n": 1}},
    {"collection_id": cid, "content": "Second", "metadata": {"n": 2}}
])
```

### Conversation Messages

```python
# Create conversation
conv_id = nebula.store_memory({
    "collection_id": cid,
    "content": "Hello!",
    "role": "assistant"
})

# Append single message
nebula.store_memory({
    "memory_id": conv_id,
    "collection_id": cid,
    "content": "I need help",
    "role": "user"
})

# Append multiple messages
nebula.store_memory({
    "memory_id": conv_id,
    "collection_id": cid,
    "content": [
        {"content": "What issue?", "role": "assistant"},
        {"content": "Login broken", "role": "user"}
    ]
})
```

### Document Upload

```python
from nebula import Memory

# Raw text (auto-chunked)
doc_id = nebula.create_document_text(
    collection_id=cid,
    raw_text="Full document content...",
    metadata={"title": "Paper"}
)

# Pre-chunked text
doc_id = nebula.create_document_chunks(
    collection_id=cid,
    chunks=["Chapter 1...", "Chapter 2..."],
    metadata={"title": "Book"}
)

# File upload (PDF, DOCX, images, audio)
doc_id = nebula.store_memory(
    Memory.from_file("document.pdf",
        collection_id=cid,
        metadata={"title": "Research"})
)
```

Supported file types: PDF, DOC, DOCX, TXT, JPEG, PNG, GIF, WebP, MP3, WAV, M4A, OGG, FLAC. Files > 5MB use presigned S3 upload (max 100MB).

## Retrieving Memories

```python
# Get by ID (includes chunks)
memory = nebula.get_memory(memory_id)
# memory.chunks -> list of chunks with .id, .content, .metadata

# List memories
memories = nebula.list_memories(
    collection_ids=["research"],
    limit=50,
    metadata_filters={"metadata.status": {"$eq": "active"}}
)

# Get conversation messages
messages = nebula.get_conversation_messages(conv_id)

# List conversations
conversations = nebula.list_conversations(limit=50)
```

## Updating

```python
# Update memory metadata (does NOT update content)
nebula.update_memory(
    memory_id=mid,
    metadata={"status": "reviewed"},
    merge_metadata=True  # merge with existing metadata
)

# Update specific chunk
nebula.update_chunk(
    chunk_id=chunk_id,
    content="Updated content",
    metadata={"edited": True}
)
```

## Deleting

```python
# Single memory
nebula.delete(memory_id)

# Multiple memories
nebula.delete([mid1, mid2, mid3])

# Single chunk
nebula.delete_chunk(chunk_id)
```

## Search

```python
results = nebula.search(
    query="machine learning",
    collection_ids=["research"],       # optional, omit for all
    limit=10,                          # optional
    offset=0,                          # optional, pagination
    filters={"metadata.topic": "AI"},  # optional
    effort="medium",                   # optional: auto/low/medium/high
    search_settings={                  # optional
        "semantic_weight": 0.8,        # default
        "fulltext_weight": 0.2,        # default
        "include_metadata": True       # return metadata in results
    }
)

for r in results:
    print(r.score)          # 0-1 relevance
    print(r.content)        # chunk content
    print(r.metadata)       # metadata dict
    print(r.memory_id)      # parent memory ID
    print(r.collection_id)  # collection ID
    print(r.created_at)     # timestamp
```

## Error Handling

```python
from nebula.exceptions import (
    NebulaException,               # base
    NebulaClientException,         # config/setup errors
    NebulaAuthenticationException, # 401
    NebulaRateLimitException,      # 429
    NebulaValidationException,     # 400
    NebulaNotFoundException,       # 404
    NebulaCollectionNotFoundException
)

try:
    results = nebula.search(query="test")
except NebulaAuthenticationException:
    print("Invalid API key")
except NebulaRateLimitException:
    print("Rate limited, retry later")
except NebulaException as e:
    print(f"Nebula error: {e}")
```

## Utilities

```python
# Health check
nebula.health_check()  # raises on failure

# Get presigned upload URL (for large files)
url_info = nebula.get_upload_url()
```
