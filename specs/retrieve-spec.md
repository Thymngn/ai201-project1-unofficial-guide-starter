# Spec: `retrieve()`

**File:** `retriever.py`
**Status:** Implemented

---

## Purpose

Given a user's natural language query, find the most relevant chunks from the
vector store using semantic similarity search. Return them ranked by relevance so
that `generate_response()` can use them as context.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's natural language question |
| `n_results` | `int` | Maximum number of chunks to return (default: `N_RESULTS` from `config.py`) |

**Output:** `list[dict]`

Each dict in the returned list must contain exactly these keys:

| Key | Type | Description |
|-----|------|-------------|
| `"text"` | `str` | The chunk text |
| `"category"` | `str` | The category this chunk came from (e.g., `"Food"`) |
| `"distance"` | `float` | Cosine distance score — lower means more similar to the query |

Results are ordered from most to least relevant (lowest to highest distance).
Returns an empty list `[]` if the collection contains no documents.

---

## Design Decisions

---

### Query approach

```
_collection.query() converts the query into an embedding inside ChromaDB using
the same all-MiniLM-L6-v2 function the chunks were stored with, then runs a
cosine-distance search. I pass:
  - query_texts = [query]
  - n_results   = n_results (default 5 from config.py)
  - include     = ["documents", "metadatas", "distances"]
"documents" gives the chunk text, "metadatas" gives the category label, and
"distances" gives the similarity score used for ranking and filtering downstream.
```

---

### Return structure

```
One item in the return list looks like:

{
    "text"     : "OWL CARD ... serves as: Official photo ID, Meal plan access card ...",
    "category" : "Food",          # pulled from metadatas[0][i]["category"]
    "distance" : 0.489,           # pulled from distances[0][i]
}

"text" comes from results["documents"][0][i], "category" from
results["metadatas"][0][i]["category"], and "distance" from
results["distances"][0][i].
```

---

### Handling the nested result structure

```
_collection.query() supports multiple queries at once, so it returns a list per
field with one entry per query. I send a single query, so I take index [0] of
each field to get the actual list of results, then zip documents, metadatas, and
distances together in rank order.
```

---

### Relevance threshold

```
retrieve() itself does NOT apply a hard distance cutoff — it returns all
n_results so the generator has the full ranked context to work with. The
relevance filter (DISTANCE_THRESHOLD) lives in generate_response() instead.
Keeping retrieval permissive and filtering at generation time means I can tune
the cutoff in one place without losing candidates here. Tradeoff: weak chunks
can appear in the returned list, but the generator drops them before building
context.
```

---

### Edge cases

```
(a) Empty collection: retrieve() returns [] immediately (checked via
    _collection.count() == 0), and the UI surfaces a "nothing loaded" message.
(b) Query matches no chunks well: chunks are still returned but with high
    distances; the generator's threshold filters them out and returns the
    fallback message. This signals a content gap or a chunking issue to revisit.
(c) Query matches chunks from multiple categories: all are returned ranked by
    distance, and the generator cites whichever categories it draws from.
```

---

## Implementation Notes

**Test query and top result returned:**

```
Query: What are the perks of the Owl Card?
Top result category: Food   (the OWL CARD section lives in food.txt)
Distance score: 0.489
Does it make sense? Yes — after raising chunk_size to 600 the full Owl Card
benefit list stays in one chunk, so the top result now contains the complete
answer instead of half of it.
```

**One thing about the query results that surprised you:**

```
When the query doesn't name a category, results come back from several
categories at once (e.g., the Owl Card query also surfaces Campus Services and
Transportation chunks). The distance ranking and the generator's threshold keep
the off-topic ones out of the final answer.
```
