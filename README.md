# Owl Search — The Unofficial Guide to FAU (Project 1)

> Owl Search is a Retrieval-Augmented Generation (RAG) assistant that answers
> plain-language questions about Florida Atlantic University using a curated set
> of FAU documents. Run it with `python app.py` and ask in the Gradio chat.

<!--
NOTE TO SELF: The reflective sections below (Failure Case Analysis, Spec
Reflection, AI Usage, and the accuracy judgments in the Evaluation Report) are
graded in your own words. The text here is a starting draft based on what the
system actually produced — review every line and rewrite it as your own before
submitting.
-->

---

## Domain

I chose general information about Florida Atlantic University as my domain. This
knowledge is valuable because it acts as a single place to look up everyday FAU
facts — academics, admissions, dorms, dining, financial aid, transportation, and
campus services — instead of clicking through the official website's many
departmental sub-sites. The information *is* public, but it's scattered: a
question like "which dorms are for upperclassmen and do they require a meal plan?"
can touch the housing site, the dining site, and a pricing PDF. Owl Search pulls
the relevant facts into one grounded answer.

---

## Document Sources

The corpus is 10 curated `.txt` files in `/docs`, each compiled from official FAU
pages (and a few external references) for one topic category. The category label
on each file is what the system cites in its answers.

| # | Source (category file) | Type | URL or file path |
|---|------------------------|------|------------------|
| 1 | `academics.txt` | Curated `.txt` from FAU official pages | https://www.fau.edu/about/academics/ |
| 2 | `admissions_general.txt` | Curated `.txt` from FAU official pages | https://www.fau.edu/admissions/freshman/how-to-apply/admissions-requirements/ |
| 3 | `campus_services.txt` | Curated `.txt` (health, counseling, victim services) | https://www.fau.edu/shs/ |
| 4 | `club_organization.txt` | Curated `.txt` from FAU involvement pages | https://www.fau.edu/involvement/clubhouse/registeredstudentorgs/ |
| 5 | `college.txt` | Curated `.txt` from FAU provost/colleges pages | https://www.fau.edu/provost/about/colleges/ |
| 6 | `cs_department.txt` | Curated `.txt` from FAU EECS pages | https://www.fau.edu/engineering/eecs/undergraduate/computer-science/ |
| 7 | `dorm.txt` | Curated `.txt` from FAU housing pages + pricing PDF | https://www.fau.edu/housing/rates/ |
| 8 | `financial_aid.txt` | Curated `.txt` from FAU financial aid pages | https://www.fau.edu/finaid/ |
| 9 | `food.txt` | Curated `.txt` from FAU dining/meal-plan pages | https://www.fau.edu/business-services/meal-plans/meal-plans-overview/ |
| 10 | `transportation.txt` | Curated `.txt` from FAU parking/transit pages | https://www.fau.edu/parking/ |

> The full list of ~38 underlying source URLs (multiple per category) is in
> [`planning.md`](./planning.md#documents).

---

## Chunking Strategy

**Chunk size:** 600 characters

**Overlap:** 100 characters

**Preprocessing:** Documents are read as UTF-8 text. The category name is derived
from the filename (`food.txt` → `Food`) and attached to every chunk as metadata.
No HTML stripping is needed since the documents are already plain text. Chunks
shorter than 50 characters (whitespace, lone headers) are discarded.

**Why these choices fit the documents:** The FAU documents are list-structured —
a dorm entry, a meal plan, or the Owl Card benefit list is a short labeled block
of a few lines. I started at 300/50 (my original plan), but that split labeled
blocks apart: the Owl Card benefit list was cut in half, so retrieval returned
only a partial answer. A 600-character window holds one complete block while
staying small enough to keep retrieval targeted, and the 100-character overlap
keeps a label-to-detail link from being severed at a boundary.

**Final chunk count:** 79 chunks across all 10 documents.

---

## Embedding Model

**Model used:** `all-MiniLM-L6-v2` (via `sentence-transformers`), producing
384-dimensional vectors. It runs locally with no API key or rate limits and is
fast on the short, factual passages in this corpus.

**Production tradeoff reflection:**
<!-- DRAFT — rewrite in your own words. -->
If I were deploying Owl Search for real users and cost wasn't a constraint, I'd
weigh a larger hosted embedding model (e.g., OpenAI `text-embedding-3-large`)
against the current local model. The larger model would likely improve accuracy
on queries where vocabulary overlaps across categories (the "meal plan" term
appears in both Food and Dorm), and its longer context window would let me embed
bigger chunks without truncation. The tradeoffs I'd accept are added per-query
latency from network calls, ongoing API cost, and a dependency on an external
service being available. For a small, mostly-English, list-style corpus like this
one, the local MiniLM model is a reasonable default; I'd only move to a hosted
model if evaluation showed retrieval misses I couldn't fix with chunking.

---

## Grounded Generation

**System prompt grounding instruction:** The system message tells the model it is
"a Florida Atlantic University informant assistant" that must "answer the user's
question using ONLY the rule excerpts provided" and "not use any outside knowledge
about documents," and to "say so clearly" when the answer isn't in the excerpts.
(See `system_prompt` in `generator.py`.)

**Structural grounding choices:** Grounding isn't enforced by the prompt alone —
there are two structural mechanisms in `generate_response()`:
1. **Relevance filtering.** Before any LLM call, chunks with a cosine distance
   above `DISTANCE_THRESHOLD` (0.65) are dropped. If nothing survives, the
   function returns a fallback message *without* calling the model, so it can't
   hallucinate from weak context.
2. **Labeled context block.** Surviving chunks are formatted as
   `[Category]\n<text>` blocks separated by `\n\n---\n\n`, with the user's
   question placed after the context. The model only ever sees retrieved text.

**How source attribution is surfaced:** The system prompt instructs the model to
name the category its answer comes from (e.g., "According to the academics
document, ..."), using the `[Category]` label embedded in each context block.

---

## Evaluation Report

The five questions from [`planning.md`](./planning.md#evaluation-plan), run
end-to-end through the implemented system (chunk size 600, top-k 5, threshold
0.65).

<!-- DRAFT accuracy/quality judgments — confirm these against your own reading. -->

| # | Question | Expected answer | System response (summarized) | Retrieval quality | Response accuracy |
|---|----------|-----------------|------------------------------|-------------------|-------------------|
| 1 | Top 5 most popular majors at FAU? | Business, Health Professions, Multi/Interdisciplinary Studies, Psychology, Biological Sciences | Said no info on *popular* majors is in the excerpts; described academics/colleges generally | Partially relevant | Inaccurate (info not in docs) |
| 2 | List the meal plan options at the dining hall | 19 / 15+$100 / 12+$100 / 7+$550 Meal plans with prices | Listed dietary options (vegan/vegetarian/GF) and the Jupiter 19/14 plans; said main plans not explicitly listed | Relevant (all Food) | Partially accurate |
| 3 | What are the perks of the Owl Card? | Photo ID, meal plan card, library card, residence hall key, event ticket, Owl Bucks | Returned the complete, correct benefit list | Relevant | Accurate |
| 4 | List dorms for upperclassmen | IRT, IVA North, IVA South, Talon Hall | Named Talon Hall and one returning-upperclassmen hall; missed IRT and the IVA halls | Partially relevant | Partially accurate |
| 5 | List some FAU Health Services | Primary & Acute Care, Sexual & Reproductive Health, Nutrition, Psychiatric, Flu Clinic, Public Health, Faculty/Staff | Gave only the SHS location and said specific services weren't listed | Partially relevant | Partially accurate |

**Overall:** 1 fully accurate, 3 partially accurate, 1 inaccurate. The accurate
and strongest cases (Owl Card, meal plans) are the ones where the answer lives in
a single tight labeled block. The weaker cases (#4, #5) are ones where the answer
is a longer list that gets split across chunks.

---

## Failure Case Analysis

<!-- DRAFT — rewrite in your own words. You may swap in question #4 instead. -->

**Question that failed:** "List some FAU Health Services" (#5).

**What the system returned:** It correctly located Student Health Services ("on
the 2nd floor of the Breezeway at the Boca Raton campus") but then said the
excerpts "do not list specific FAU Health Services beyond the location" — even
though the full list (Primary & Acute Health Care, Sexual & Reproductive Health,
Nutrition Services, Psychiatric Services, Flu Clinic, Public Health Advisory,
Faculty/Staff services) is present in `campus_services.txt`.

**Root cause (tied to a pipeline stage):** Retrieval, driven by chunking. The
bulleted services list and the SHS overview text sit close together in the
source file, but character-based chunking placed the bullet list in a different
600-character chunk than the overview. For the generic query "List some FAU
Health Services," the overview chunk (which repeats "health services" in prose)
scored a closer cosine distance than the bullet-list chunk, so the bullet chunk
fell outside the top-5 and never reached the model. The model answered faithfully
from the context it was given — it just wasn't given the list.

**What I would change to fix it:** Move to list-aware chunking that keeps a
header and its bullets in one chunk, or raise top-k so the bullet chunk is more
likely to be included. A targeted fix would be to detect bulleted blocks during
ingestion and emit them as whole chunks regardless of the 600-character window.

---

## Spec Reflection

<!-- DRAFT — rewrite in your own words with your own examples. -->

**One way the spec helped during implementation:** Writing the
`generate-reponse-spec.md` design decisions before coding forced me to decide how
to handle weak chunks *up front*. Because I'd already committed to a distance
threshold and a "return a fallback without calling the LLM if nothing survives"
rule, the grounding behavior was built in from the first version instead of being
patched on later. The spec's input/output contract also kept the three functions
compatible — `retrieve()` returns exactly the keys `generate_response()` expects.

**One way the implementation diverged from the spec, and why:** My original spec
set the distance threshold at 0.5 and chunk size at 300. In testing, the Owl Card
benefit chunk scored ~0.64 and was being filtered out, so the answer came back
incomplete. I diverged by raising chunk size to 600 (which made related facts
land in one chunk and dropped that distance to ~0.49) and raising the threshold to
0.65. The divergence came directly from evaluation evidence, not a guess — the
spec was the starting hypothesis and the test results corrected it.

---

## AI Usage

<!-- DRAFT — these must describe YOUR actual AI usage. Replace with real instances. -->

**Instance 1 — Tuning retrieval/generation parameters**

- *What I gave the AI:* My running app plus the complaint that answers were
  "not quite there," and the three core files (`config.py`, `retriever.py`,
  `generator.py`).
- *What it produced:* It ran my five evaluation queries, printed the cosine
  distances, and showed that the relevant Owl Card chunk (distance ~0.64) was
  being filtered out by my 0.5 threshold. It recommended chunk size 600, overlap
  100, top-k 5, and threshold 0.65, then re-ingested and re-tested.
- *What I changed or overrode:* [TODO — fill in what you accepted, rejected, or
  adjusted. e.g., did you keep all four values, or tune any yourself afterward?]

**Instance 2 — [TODO: your second real instance]**

- *What I gave the AI:*
- *What it produced:*
- *What I changed or overrode:*
