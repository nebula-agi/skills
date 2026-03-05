# Search and Metadata Filtering Reference

## Search Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | string | Search query (natural language works best) |
| `collection_ids` | string[] | Collections to search (omit for all) |
| `limit` | number | Max results to return |
| `offset` | number | Pagination offset |
| `filters` | object | Metadata filters (see below) |
| `effort` | string | `auto`/`low`/`medium`/`high` (default: `auto`) |
| `search_settings` | object | Hybrid weights and options (see below) |

## Hybrid Search Weights

Control semantic vs keyword balance via `search_settings`:

| Setting | Default | Description |
|---------|---------|-------------|
| `semantic_weight` | 0.8 | Weight for vector/meaning-based search |
| `fulltext_weight` | 0.2 | Weight for keyword/full-text search |
| `include_metadata` | false | Return metadata in results |

**Tuning guidance:**
- `0.8 / 0.2` -- default, prefer semantic meaning
- `0.5 / 0.5` -- balanced semantic + keyword
- `0.2 / 0.8` -- prefer exact keyword matches

## Search Effort

Controls traversal compute and exploration depth:

| Level | Use case |
|-------|----------|
| `auto` | Adapts based on query complexity (default) |
| `low` | Fast, fewer candidates explored |
| `medium` | Balanced coverage and performance |
| `high` | Maximum coverage, slower |

## Search Result Fields

| Field | Type | Description |
|-------|------|-------------|
| `score` | float | Relevance score, 0-1 (higher = better) |
| `content` | string | Chunk content |
| `metadata` | object | Custom metadata (if `include_metadata` is true) |
| `memory_id` | string | Parent memory ID |
| `collection_id` | string | Collection ID |
| `created_at` | string | ISO timestamp |

## Authority Scores

Set `authority` (0-1) when storing memories to prioritize content in search:

| Score | Use case |
|-------|----------|
| 0.9-1.0 | Verified facts, official docs |
| 0.7-0.9 | Reliable sources |
| 0.5-0.7 | General content (default) |
| 0.3-0.5 | Uncertain information |
| 0.0-0.3 | Low quality |

```python
nebula.store_memory({
    "collection_id": cid,
    "content": "Official API docs...",
    "authority": 0.95
})
```

Per-message authority in conversations:

```python
nebula.store_memory({
    "memory_id": conv_id,
    "collection_id": cid,
    "content": "Verified answer",
    "role": "assistant",
    "authority": 0.9
})
```

## Metadata Filter Operators

Target metadata fields with the `metadata.` prefix in REST API. SDKs accept both prefixed and unprefixed keys.

| Operator | Description | Example |
|----------|-------------|---------|
| `$eq` | Equals (default for bare values) | `{"status": "active"}` |
| `$ne` | Not equals | `{"status": {"$ne": "archived"}}` |
| `$gt` | Greater than | `{"score": {"$gt": 80}}` |
| `$gte` | Greater than or equal | `{"priority": {"$gte": 7}}` |
| `$lt` | Less than | `{"age": {"$lt": 30}}` |
| `$lte` | Less than or equal | `{"count": {"$lte": 100}}` |
| `$in` | Value in list | `{"status": {"$in": ["a", "b"]}}` |
| `$nin` | Not in list | `{"status": {"$nin": ["x", "y"]}}` |
| `$like` | Pattern match (case-sensitive) | `{"title": {"$like": "Important%"}}` |
| `$ilike` | Pattern match (case-insensitive) | `{"email": {"$ilike": "%@company.com"}}` |
| `$overlap` | Array has any of | `{"tags": {"$overlap": ["urgent"]}}` |
| `$contains` | Array has all of | `{"skills": {"$contains": ["python", "ml"]}}` |
| `$and` | All conditions match | `{"$and": [{...}, {...}]}` |
| `$or` | Any condition matches | `{"$or": [{...}, {...}]}` |

Use `%` as wildcard in `$like`/`$ilike`. Nested properties supported: `metadata.user.profile.age`.

## Common Filter Patterns

### Date Ranges

```python
filters = {
    "$and": [
        {"metadata.created_at": {"$gte": "2024-01-01"}},
        {"metadata.created_at": {"$lte": "2024-12-31"}}
    ]
}
```

### Multi-Status

```python
# Include specific
filters = {"metadata.status": {"$in": ["pending", "active", "review"]}}

# Exclude specific
filters = {"metadata.status": {"$nin": ["archived", "deleted"]}}
```

### Array Operations

```python
# Any of these tags
filters = {"metadata.tags": {"$overlap": ["urgent", "important"]}}

# Must have ALL of these skills
filters = {"metadata.skills": {"$contains": ["python", "ml"]}}
```

### Combined Logical

```python
filters = {
    "$and": [
        {"metadata.priority": {"$gte": 7}},
        {"$or": [
            {"metadata.status": "active"},
            {"metadata.status": "pending"}
        ]}
    ]
}
```
