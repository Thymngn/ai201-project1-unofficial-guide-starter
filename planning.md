# Project 1 Planning: The Unofficial Guide

> Write this document before you write any pipeline code.
> Your spec and architecture diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Update the Retrieval Approach and Chunking Strategy sections if you change your approach during implementation.
> Update this file before starting any stretch features.

---

## Domain

<!-- What domain did you choose? Why is this knowledge valuable and hard to find through official channels? -->
I choose general information about Florida Atlantic University as my domain. This knowledge will serves as an one stop to search general information about Florida Atlantic University rather than exploring many pages. The official website has a lot of sub links which making research a longer process.
---

## Documents

<!-- List your specific sources: URLs, subreddit names, forum threads, or file descriptions.
     Aim for at least 10 sources that together cover different subtopics or perspectives within your domain. -->

| # | Source | Description | URL or location |
|---|--------|-------------|-----------------|
| 1 |FAU Academics Overview|Overview of FAU academic structure, programs, and general education requirements| https://www.fau.edu/about/academics/
| 2 |FAU Degree Programs|List and details of undergraduate and graduate academic programs offered at FAU| https://www.fau.edu/programs/
| 3 |US News FAU Academics|External ranking and summary of FAU academic reputation and program strengths| https://www.usnews.com/best-colleges/florida-atlantic-university-1481/academics
| 4 |Freshman Admissions Requirements|Requirements and criteria for incoming freshman applicants including GPA and test scores|https://www.fau.edu/admissions/freshman/how-to-apply/admissions-requirements/
| 5 |FAU Admissions Catalog |Official admissions policies and university catalog information for enrollment|https://www.fau.edu/registrar/university-catalog/catalog/admissions/
| 6 |US News Admissions Overview|Summary of FAU admissions process, acceptance rates, and application competitiveness|https://www.usnews.com/best-colleges/florida-atlantic-university-1481/applying
| 7 |Student Health Services |Campus health services including medical care, vaccinations, and wellness support|https://www.fau.edu/shs/
| 8 |Counseling Services| Mental health counseling, wellness programs, and psychological support for students|https://www.fau.edu/counseling/
| 9 |Campus Life Overview|Overview of student life, campus activities, housing, and engagement opportunities|https://www.fau.edu/about/campus-life
| 10 |Victim Services Resources|Campus resources for safety, victim support, and emergency assistance services|https://www.fau.edu/dean/victimservices/campusresources/
| 11 |Student Organizations Directory |List of registered student organizations and clubs available at FAU|https://www.fau.edu/involvement/clubhouse/registeredstudentorgs/
| 12 |Campus Life Activities|Student engagement, events, and general campus involvement opportunities|https://www.fau.edu/about/campus-life
| 13 |Arts & Letters Graduate Clubs| Graduate-level clubs and student organizations within Arts and Letters programs |https://www.fau.edu/artsandletters/graduate-studies/clubs-and-organizations/
| 14 |Engineering Student Clubs|Engineering-specific student organizations, competitions, and technical clubs |https://www.fau.edu/engineering/case/clubs/
| 15 |FAU Colleges Overview |Overview of all academic colleges and schools within Florida Atlantic University|https://www.fau.edu/provost/about/colleges/
| 16 |Florida Board of Governors Profile| Official statewide overview of FAU including mission and institutional data|https://www.flbog.edu/university/florida-atlantic-university/
| 17 |FAU Academic Structure|General information about FAU academic organization and degree pathways|https://www.fau.edu/about/academics/
| 18 |Engineering EECS Department|Information about Electrical Engineering and Computer Science department|https://www.fau.edu/engineering/eecs/
| 19 |Computer Science Undergraduate Program|Details on CS major requirements, curriculum, and degree structure |https://www.fau.edu/engineering/eecs/undergraduate/computer-science/
| 20 |CS Graduate Program |Master’s program overview in Computer Science including admission and structure|https://www.fau.edu/engineering/eecs/graduate/ms/computer-science/
| 21 |CS Graduate Courses|Detailed list and descriptions of graduate-level CS courses|https://www.fau.edu/engineering/eecs/graduate/ms/computer-science/courses/
| 22 |Housing Rates|On-campus housing pricing, dorm costs, and residence hall rate information|https://www.fau.edu/housing/rates/
| 23 |Dorm Overview (External)|Third-party overview of FAU dorms, student housing experience, and reviews|https://www.mydorm.com/colleges/florida-atlantic-university/
| 24 |Campus Life Overview (Duplicate Source)|General campus life information including housing, dining, and student engagement|https://www.fau.edu/about/campus-life
| 25 |Housing Pricing PDF|Official housing cost breakdown for specific residence halls and academic year pricing|https://www.fau.edu/housing/pdf/2026-2027-fau-housing-talon-hall-pricing.pdf
| 26 |Financial Aid Office|Official financial aid resources including grants, loans, and scholarships|https://www.fau.edu/finaid/
| 27 |Scholarships Overview|External breakdown of scholarships and financial aid opportunities at FAU|https://www.niche.com/colleges/florida-atlantic-university/scholarships-financial-aid/
| 28 |Financial Aid Guide|Guide explaining financial aid options and eligibility at FAU|https://www.collegevine.com/faq/146841/what-financial-aid-opportunities-are-available-at-florida-atlantic-university
| 29 |Financial Aid Summary|Overview of FAFSA, aid packages, and student funding options|https://www.thecollegemonk.com/colleges/florida-atlantic-university/financial-aid
| 30 |Campus Life Overview (Duplicate Source)|Student life information including housing, dining, and campus engagement|https://www.fau.edu/about/campus-life
| 31 |Residential Meal Plans|Dining plans for students living on campus and meal plan options|https://dineoncampus.com/fau/residential-meal-plans
| 32 |Meal Plan Overview |Explanation of FAU meal plan options, pricing, and structure|https://www.fau.edu/business-services/meal-plans/meal-plans-overview/
| 32 |Meal Plan FAQ |Frequently asked questions about meal plans and dining services|https://www.fau.edu/business-services/meal-plans/faq/
| 34 |Parking Services|Campus parking rules, permits, enforcement, and regulations |https://www.fau.edu/parking/
| 35 |Transportation Resources|Student transportation options including shuttles and commuting guidance|https://www.fau.edu/newstudent/family/resources/transportation/
| 36 |Parking Operations|Parking enforcement, permits, and operational services |https://www.fau.edu/parking/services/
| 37 |Campus Shuttle System|FAU shuttle routes and campus transportation services|https://www.fau.edu/parking/bocacampusshuttle/
| 38 |Public Transit Options|Palm Tran and public transportation access near FAU campuses|https://www.fau.edu/parking/palmtran/


---

## Chunking Strategy

<!-- How will you split documents into chunks?
     State your chunk size (in tokens or characters), overlap size, and explain why those
     numbers fit the structure of your documents.
     A review-heavy corpus warrants different chunking than a long FAQ. -->

**Chunk size:**
300 character. 
**Overlap:**
50 words

**Reasoning:**
The documents are organized mostly in list format, where individual facts (e.g., a meal plan price,
a dorm name, a club name) fit within 1–3 sentences.

A 300-character chunk captures one to two
distinct facts without pulling in unrelated details from the next list item.
A larger chunk (e.g., 1,000 characters) would blur multiple distinct facts together, making retrieval less precise — for example, mixing dorm amenity details with pricing when a user only asks about one.

The 50-character overlap (roughly 8–12 words) ensures that facts which span a chunk boundary —
such as a building name on one line and its phone number on the next — aren't split completely.

This overlap is intentionally small because the documents are short, factual, and list-like;
large overlaps would create near-duplicate chunks that waste retrieval slots.
---

## Retrieval Approach

<!-- Which embedding model are you using (e.g., all-MiniLM-L6-v2 via sentence-transformers)?
     How many chunks will you retrieve per query (top-k)?
     If you were deploying this for real users and cost wasn't a constraint, what tradeoffs
     would you weigh in choosing a different embedding model — context length, multilingual
     support, accuracy on domain-specific text, latency? -->

**Embedding model:**
"all-MiniLM-L6-v2"

**Top-k:**
3
With 300-character chunks, three chunks give the generator roughly 900 characters (~180–240 tokens) of context — enough to answer a focused factual question without exceeding context limits or introducing irrelevant noise.

**Production tradeoff reflection:**

---

## Evaluation Plan

<!-- List your 5 test questions with their expected correct answers.
     Questions should be specific enough that you can judge whether the system's response
     is right or wrong. "What are good dining halls?" is too vague.
     "What do students say about wait times at [dining hall name] during lunch?" is testable. -->

| # | Question | Expected answer |
|---|----------|-----------------|
| 1 |What top 5 most popular major at Florida Atlantic University?|1. Business, Management, Marketing, and Related Support Services, Health Professions and Related Programs, Multi/Interdisciplinary Studies, Psychology, Biological and Biomedical Sciences|
| 2 |List the meal plan options at the dining hallPlan Options (per semester):
  1. 19 Meals/Week: Price: $2,248.50/semester, Best for students with a full, busy schedule,
  2. 15 Meals/Week + $100 Flex Bucks, Price: $2,123.50/semester, Balanced plan: structured meals + flexible campus dining spending
  3. 12 Meals/Week + $100 Flex Bucks, Price: $1,902.50/semester, Good for students with lighter dining needs
  4. 7 Meals/Week + $550 Flex Bucks, Price: $2,225.50/semester, Ideal for students who eat off-campus often but want on-campus convenience
| 3 |What are the perks of Owl Card|Meal plan access card, Library card, Residence hall key, Ticket to many FAU events (including sporting events),  Owl Bucks debit card (NOT a regular banking debit card)
| 4 |List dorms for upperclassmen|INDIAN RIVER TOWERS (IRT),INNOVATION VILLAGE APARTMENTS NORTH (IVA North), INNOVATION VILLAGE APARTMENTS SOUTH (IVA South), TALON HALL (NEW — Opening Fall 2026)
| 5 |List some FAU Health Services|Primary & Acute Health Care, Sexual & Reproductive Health, Nutrition Services, Psychiatric Services, Flu Clinic, Public Health, Faculty and Staff services|

---

## Anticipated Challenges

<!-- What could go wrong? Name at least two specific risks with reasoning.
     Consider: noisy or inconsistent documents, missing source attribution, off-topic
     retrieval, chunks that split key information across boundaries. -->

1. Chunk boundary splits on related facts:
List-structured documents often have a label on one line (e.g., a dorm name) and its details on
the next several lines. A fixed-character chunker may split the label from its details, causing
a retrieved chunk to lack context. The 50-character overlap reduces this but won't eliminate it.
Mitigation: inspect chunk boundaries during testing and adjust overlap or chunk size if specific
facts are consistently split.
2. Off-topic retrieval due to overlapping vocabulary:
Several FAU topics share vocabulary — for example, "meal plan" appears in both the food and the
dorm files, and "parking" appears in transportation and financial aid (permit fees). A question
about meal plan costs could retrieve chunks about housing requirements (which mention meal plans)
instead of the pricing chunks. Mitigation: increase top-k to 5 during evaluation and verify that
at least 3 of the 5 chunks are on-topic. If off-topic retrieval is frequent, consider
metadata-filtered retrieval (e.g., filter by source file).

---

## Architecture

┌─────────────────────────────────────────────────────────────────────────┐
│                        RAG PIPELINE — FAU GUIDE                         │
└─────────────────────────────────────────────────────────────────────────┘

 ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────────┐
 │  1. DOCUMENT     │    │  2. CHUNKING     │    │  3. EMBEDDING +      │
 │     INGESTION    │───▶│                  │───▶│     VECTOR STORE     │
 │                  │    │                  │    │                      │
 │ Tool: Python     │    │ Tool: Python     │    │ Embedding:           │
 │ (open / read     │    │ (custom          │    │  sentence-transformers│
 │  .txt files)     │    │  character-based │    │  all-MiniLM-L6-v2   │
 │                  │    │  splitter)       │    │                      │
 │ Input:           │    │                  │    │ Vector Store:        │
 │  10 .txt files   │    │ Chunk size: 300  │    │  ChromaDB   │
 │  (academics,     │    │ Overlap:    50   │    │  (in-memory or       │
 │   admissions,    │    │ chars each       │    │   persisted)         │
 │   campus_svcs,   │    │                  │    │                      │
 │   clubs, college,│    │ Output:          │    │ Output:              │
 │   cs_dept, dorm, │    │  list of text    │    │  indexed 384-dim     │
 │   finaid, food,  │    │  chunks with     │    │  embeddings stored   │
 │   transport)     │    │  metadata (src)  │    │  with chunk text     │
 └──────────────────┘    └──────────────────┘    └──────────────────────┘
                                                           │
                                                           ▼
 ┌──────────────────┐    ┌──────────────────────────────────────────────┐
 │  5. GENERATION   │    │  4. RETRIEVAL                                │
 │                  │◀───│                                             │
 │ Tool: Grok       │    │ Tool: sentence-transformers + vector store   │
 │                  │    |                                              │
 │                  │    │ - Embed user query with all-MiniLM-L6-v2     │
 │ Input:           │    │ - Cosine similarity search                   │
 │  user query +    │    │ - Return top-k = 3 most similar chunks       │
 │  top-3 chunks    │    │                                              │
 │  as context      │    │ Output: 3 text chunks + source metadata      │
 │                  │    └──────────────────────────────────────────────┘
 │ Output:          │
 │  grounded answer │    ┌──────────────────┐
 │  citing source   │    │  USER INTERFACE  │
 │  file names      │◀──▶│                 │
 └──────────────────┘    │ Tool: Gradio     │
                         │ or CLI (input /  │
                         │  output loop)    │
                         └──────────────────┘

---

## AI Tool Plan

<!-- For each part of the pipeline below, describe:
     - Which AI tool you plan to use (Claude, Copilot, ChatGPT, etc.)
     - What you'll give it as input (which sections of this planning.md, which requirements)
     - What you expect it to produce
     - How you'll verify the output matches your spec

     "I'll give Claude my Chunking Strategy section and ask it to implement chunk_text()
     with my specified chunk size and overlap" is a plan. -->

     Input given to Claude: The 10 topic categories (academics, admissions, campus services, clubs, college, CS department, dorm, financial aid, food, transportation) + "Florida Atlantic University" as the domain.

     What Claude produced: Web searches across 39 FAU sources and generated 10 structured .txt files with factual content, section headers, and source URLs at the top of each file.

     Verification: Each file was reviewed against the cited source URLs to confirm accuracy.

**Milestone 3 — Ingestion and chunking:**
     - Tool: Groq
     - Input to : This planning.md (Chunking Strategy section) + the 10 .txt files
     - Prompt template: "Implement an ingest_and_chunk() function in Python that reads all .txt files
     from a given directory, splits each file's text into chunks of 300 characters with 50-character
     overlap, and returns a list of dicts with keys text, source (filename), and chunk_id."
     - Expected output: A working ingest.py module with ingest_and_chunk(directory: str) -> list[dict]
     - Verification: Print first 5 chunks and confirm chunk sizes and overlap are correct; confirm
     source filename is correctly attached to each chunk.
**Milestone 4 — Embedding and retrieval:**
     - Tool: Claude
     - Input to Claude: This planning.md (Retrieval Approach section) + the chunk list schema from M3
     - Prompt template: "Using sentence-transformers all-MiniLM-L6-v2 and ChromaDB, implement
     build_index(chunks) that embeds and stores all chunks, and retrieve(query, k=3) that returns
     the top-k most similar chunks to a query string. Include the source filename in returned results."
     - Expected output: retrieval.py with build_index() and retrieve() functions
     - Verification: Run the 5 evaluation queries from this planning doc and manually check that
     returned chunks contain the expected answer content.

**Milestone 5 — Generation and interface:**
     - Tool: Claude
     - Input to Claude: This planning.md (Evaluation Plan section) + retrieval output schema from M4
     - Prompt template: "Using the Anthropic Python SDK, implement generate_answer(query, chunks)
     that sends the user query and top-3 retrieved chunks as context to groq and returns a grounded answer. Then build a simple Streamlit or CLI interface that accepts user questions, calls retrieve() and generate_answer(), and prints the answer with source citations."
     - Expected output: generator.py and app.py (or main.py for CLI)
     - Verification: Run all 5 evaluation questions end-to-end and score responses as correct /
     partially correct / incorrect based on expected answers in the Evaluation Plan table.
