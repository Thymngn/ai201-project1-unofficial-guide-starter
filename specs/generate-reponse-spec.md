# Spec: `generate_response()`

**File:** `generator.py`
**Status:** Implemented

---

## Purpose

Given a user query and a list of retrieved FAU chunks, generate a response that
directly answers the question using only the retrieved text as context. The
response must be grounded — it should not draw on the model's general knowledge
of FAU or universities, only on what was retrieved.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's original question |
| `retrieved_chunks` | `list[dict]` | Ranked list of chunks from `retrieve()`, each with `"text"`, `"category"`, and `"distance"` |

**Output:** `str`

A plain string containing the response to show the user. The response should:
- Answer the question using only the retrieved text
- Identify which category the answer comes from
- Acknowledge clearly when the answer is not found in the loaded documents

Returns a fallback string (not an error) when `retrieved_chunks` is empty or when
every chunk is filtered out as too weak.

---

## Design Decisions

---

### Context formatting

```
Each chunk is rendered as a labeled block: the category in square brackets on
its own line, followed by the chunk text. Blocks are separated by a structural
delimiter (\n\n---\n\n) so the LLM can tell where one excerpt ends and the next
begins. Distance scores are used to filter and order chunks before formatting,
but they are not shown to the model — only the category label and text are.
```

---

### System prompt — grounding instruction

```
You are a Florida Atlantic University informant assistant. Answer the user's
question using ONLY the excerpts provided below. Do not use any outside
knowledge about FAU. If the answer cannot be found in the excerpts, say so
explicitly.
```

---

### System prompt — citation instruction

```
Always identify which category your answer comes from, using the category label
provided in the context (e.g. "According to the academics document, ...").
If chunks from multiple categories are relevant, cite each one.
```

---

### Fallback behavior

```
When no chunks are passed in:
  "I couldn't find anything relevant in the loaded documents. Try rephrasing
   your question — or check that your ingestion pipeline is working."

When chunks exist but all are filtered out by the distance threshold:
  "I couldn't find a confident answer to that question in the loaded documents."
```

---

### Handling low-relevance chunks

```
Filter out chunks with distance above DISTANCE_THRESHOLD (0.65) before building
context. Lower distance = more similar, so anything above the threshold is
likely noise for this embedding model and corpus. I started at 0.5, but that
cutoff discarded a genuinely relevant chunk (the Owl Card benefit list scored
~0.64), so I raised it to 0.65. Tradeoff: a stricter cutoff risks an empty
context for niche queries, while a looser one risks the model latching onto a
weak chunk. If filtering leaves no chunks, return the fallback message directly
without making an API call.
```

---

### Message structure

```
System message: grounding instruction + citation instruction (combined). The
system message sets the rules — the model must follow them for the whole
conversation.

User message: the formatted context block first, then the user's query. Putting
context before the question keeps the model grounded before it sees what's being
asked.

  system: "You are a Florida Atlantic University informant assistant. Answer
           using ONLY the excerpts provided. Cite the category..."

  user:   "Rule excerpts:\n\n[Food]\n...\n\n---\n\n[Dorm]\n...\n\n
           Question: What are the perks of the Owl Card?"
```

---

## Implementation Notes

**Test query and response:**

```
Query: What are the perks of the Owl Card?
Response: "According to the OWL CARD excerpt, the perks of the Owl Card are that
          it serves as: Official photo ID, Meal plan access card, Library card,
          Residence hall key, Ticket to many FAU events, Owl Bucks debit card..."
Correctly grounded? Yes — every item matches the food.txt OWL CARD section.
Cited the right category? Yes — it names the OWL CARD / Food excerpt.
```

**One thing you changed from your original spec after seeing the actual output:**

```
The original DISTANCE_THRESHOLD of 0.5 was too strict: the Owl Card benefit
chunk scored ~0.64 and was being filtered out entirely, so the model returned an
incomplete answer. After raising chunk_size to 600 (which lowered that distance
to ~0.49) and raising the threshold to 0.65, the full answer comes through.
```
