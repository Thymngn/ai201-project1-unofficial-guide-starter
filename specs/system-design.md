# Owl Search — System Design

**Status:** Complete
**Last updated:** June 2026

---

## Problem Statement

Florida Atlantic University's official website is comprehensive but spread across
dozens of departmental sub-sites. A prospective or current student who wants a
single fact — a meal plan price, which dorms are for upperclassmen, what the Owl
Card is good for — often has to click through several pages across different
departments to find it, if they can find it at all.

Owl Search solves this by making FAU's general information queryable. A user asks
a plain-language question and gets a specific answer drawn from a curated set of
FAU documents, with the source category surfaced so the answer can be verified.

The core constraint: answers must be grounded in the loaded FAU documents, not in
what the language model already knows about FAU or universities in general. A
confident wrong answer is worse than an honest "I couldn't find that."

---

## Architecture

Owl Search is a RAG (Retrieval-Augmented Generation) pipeline with four components:

```
User query
    │
    ▼
[1] INGEST          ──► FAU document text is chunked and stored once at startup
    ingest.py
    │
    ▼
[2] RETRIEVE        ──► Query is embedded and matched against stored chunks
    retriever.py         via semantic similarity search
    │
    ▼
[3] GENERATE        ──► Retrieved chunks are passed as context to an LLM,
    generator.py         which produces a grounded, category-attributed answer
    │
    ▼
[4] UI              ──► Gradio chat interface serves the response to the user
    app.py
```

All four components are implemented.

---

## Technical Decisions

### Embedding model: `all-MiniLM-L6-v2`

A lightweight sentence-transformers model that runs locally with no API key or
rate limits. It maps text to 384-dimensional vectors, with good performance on
short to medium passages. Tradeoffs accepted: lower accuracy than larger models
(e.g., OpenAI's `text-embedding-3-large`), but no cost and no latency from network
calls — a good fit for short, factual FAU documents.

### Vector store: ChromaDB (persistent)

ChromaDB runs locally and persists its index to `./chroma_db` on disk. This means
ingestion only has to happen once — subsequent startups skip it if the collection
is already populated. Similarity metric is cosine distance (configured in
`retriever.py`). To re-ingest after changing the chunking strategy, delete the
`./chroma_db` folder and restart.

### LLM: Groq (`llama-3.3-70b-versatile`)

Groq provides fast inference on Llama 3.3 70B via a free API tier. The model is
capable enough to follow grounding instructions reliably when they're written
clearly. The API key is loaded from `.env` and accessed via `config.py`.

### Distance metric: cosine similarity

Lower distance = more similar. Results from `_collection.query()` include a
`distances` field — a distance of 0 means identical. For this embedding model and
this corpus, observed relevant matches fall roughly below ~0.65, so
`generate_response()` filters out chunks above that threshold before building
context (see `DISTANCE_THRESHOLD` in `generator.py`).

---

## Configuration Summary

These are the values the implemented system runs with (see `config.py`,
`ingest.py`, and `generator.py`):

| Parameter | Value | Where |
|-----------|-------|-------|
| Chunk size | 600 characters | `ingest.py` |
| Chunk overlap | 100 characters | `ingest.py` |
| Minimum chunk length | 50 characters | `ingest.py` |
| Embedding model | `all-MiniLM-L6-v2` | `config.py` |
| Top-k retrieved | 5 | `config.py` (`N_RESULTS`) |
| Distance threshold | 0.65 | `generator.py` |
| LLM | `llama-3.3-70b-versatile` (Groq) | `config.py` |
| Documents | 10 `.txt` files in `/docs` | `config.py` (`DOCS_PATH`) |
| Total chunks stored | 79 | produced by ingestion |

---

## What Is Built

| File | Status | What it does |
|------|--------|-------------|
| `app.py` |  Complete | Gradio UI, startup orchestration, ingestion trigger |
| `config.py` |  Complete | Central configuration for models, paths, retrieval params |
| `ingest.py` — `load_documents()` |  Complete | Reads all `.txt` files from `/docs`, returns structured dicts |
| `ingest.py` — `chunk_document()` |  Complete | Splits a document into overlapping character-based chunks |
| `retriever.py` — ChromaDB init |  Complete | Client, collection, and embedding function are initialized |
| `retriever.py` — `embed_and_store()` |  Complete | Embeds chunks and adds them to the collection |
| `retriever.py` — `retrieve()` |  Complete | Runs semantic search for a query, returns ranked chunks |
| `generator.py` — Groq client init |  Complete | Client is initialized and ready |
| `generator.py` — `generate_response()` |  Complete | Filters weak chunks, formats context, generates a grounded answer |

---

## Component Specs

The specs for the three core functions are in this directory:

- [`chunks-document-spec.md`](./chunks-document-spec.md)
- [`retrieve-spec.md`](./retrieve-spec.md)
- [`generate-reponse-spec.md`](./generate-reponse-spec.md)

Each spec has a complete input/output contract and the design decisions behind
the implementation.
