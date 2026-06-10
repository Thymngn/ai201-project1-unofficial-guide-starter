# Spec: `chunk_document()`

**File:** `ingest.py`
**Status:** Implemented

---

## Purpose

Split a single FAU document into smaller chunks suitable for embedding and
semantic retrieval. Each chunk should carry enough context to be meaningful on
its own when retrieved in response to a user query.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `text` | `str` | The full text of an FAU document |
| `category` | `str` | The category this document belongs to (e.g., `"Food"`, `"Dorm"`) |

**Output:** `list[dict]`

Each dict in the returned list contains exactly these keys:

| Key | Type | Description |
|-----|------|-------------|
| `"text"` | `str` | The chunk text |
| `"category"` | `str` | The category name (passed through from `category`) |
| `"chunk_id"` | `str` | A unique identifier for this chunk (e.g., `"food_0"`, `"food_1"`) |

Returns an empty list `[]` if the input text is empty or produces no valid chunks.

---

## Design Decisions

---

### Splitting approach

```
Character-based sliding window. The document text is stepped through in
fixed-size windows of `chunk_size` characters, advancing by
(chunk_size - overlap) on each step so adjacent chunks share a small
region of text at their boundary.
```

---

### Chunk size

```
600 characters. The FAU documents are list-structured — a dorm entry, a meal
plan, or the Owl Card benefit list is usually a short labeled block of several
lines. The initial value of 300 characters split these blocks apart (e.g., the
Owl Card benefit list was cut in half), so retrieval returned partial answers.
Raising the window to 600 characters keeps one complete labeled block together
in a single chunk while still being small enough to keep retrieval targeted.
```

---

### Overlap

```
100 characters of overlap between adjacent chunks. If a labeled block falls
exactly on a chunk boundary, neither chunk alone contains the full block.
Overlap duplicates the tail of each chunk at the start of the next, so
boundary-spanning content can still be retrieved intact. 100 characters is
roughly one to two short lines — enough to preserve a label-to-detail link
without significantly bloating the database.
```

---

### Minimum chunk length

```
50 characters. Chunks shorter than this are discarded. Very short segments
typically contain only whitespace, section headers, or punctuation — content
that has no semantic signal and would just add noise to the vector database.
```

---

### Rationale

```
FAU's documents pack distinct facts into short labeled blocks (a price, a dorm
name, a service), so chunks that hold one complete block outperform both very
small chunks (which fragment a block) and very large chunks (which merge
unrelated blocks and blur retrieval). A 600-character window is typically one
complete block — the right unit of retrieval for questions like "What are the
perks of the Owl Card?" or "List the meal plan options."
```

---

### Known limitations

```
Character-based splitting is indifferent to sentence and list boundaries. A
chunk can still begin mid-sentence or split a long bulleted list across two
chunks even with overlap, if the list is longer than `chunk_size`. This showed
up in testing: the full FAU Health Services bullet list and the complete set of
upperclassmen dorms were split across chunks, so a single retrieved chunk
sometimes contained only part of the answer. A list-aware or sentence-aware
splitter would handle these cases better, at the cost of more implementation
complexity.
```

---

## Implementation Notes

**Actual chunk count produced across all 10 FAU documents:**

```
79
```

**One thing that surprised you or didn't match your expectations:**

```
Lower chunk count than I thought — I expected closer to the maximum across all
10 documents. The documents are short and list-like, so a 600-character window
covers a lot of each file per chunk.
```
