---
title: MasrData RAG proof of concept
status: draft
created: 2026-10-11
updated: 2026-10-11
---

# PRD: MasrData RAG proof of concept

## 1. Purpose and outcome

Build a web-based proof of concept over client-provided Egyptian CSV and JSON datasets across a five-month delivery period. A user asks a question in formal Arabic or English. The system searches the relevant subject's ingested data, performs lookup, ranking, aggregation, or supported time comparison, and returns an answer with traceable citations. An LLM may explain or synthesize retrieved text when that adds value; it must not supply unsupported facts.

This document defines a first-week POC, an MVP due by the end of month 2 [ASSUMPTION: “by month 2” means the end of week 8], and continued validated onboarding through month 5. It is not a public launch. The first-week POC is a narrow working slice; the MVP carries the agreed single-subject question types. Cross-dataset questions remain deferred. The client has confirmed the source formats, answer types, external API permission, web UI, languages, and additive ingestion. The remaining decisions concern the actual CSV/JSON schema and reconciliation rules, citation granularity, update semantics, and measurable acceptance targets (§10). The [estimate](estimate-masrdata-rag-poc.md) remains preliminary until sample files are inspected.

## 2. Users and jobs

- **Primary user [ASSUMPTION]:** An internal analyst or client reviewer asking questions about available Egypt datasets in formal Arabic or English through a web UI.
- Find a fact, list, ranking, or simple comparison within a covered subject without opening each source file.
- Understand the period, geography, measure, and source behind an answer.
- Know when the system has no dataset for a subject, or when a dataset exists but cannot answer the particular question.
- **Data operator [ASSUMPTION]:** A team member adds and checks new client CSV/JSON data in repeated ingestion cycles.
- Add further subjects over the five-month period and verify which datasets and versions are available for questions after each ingestion cycle.

## 3. Evidence available today

Only metadata JSON files are currently present in `data/`. They describe CSV datasets and other artifacts but do not contain the underlying data rows. The client will provide CSV and JSON data reports for the POC. Some metadata also names workbooks and PDFs; those are outside the confirmed POC intake unless the client later changes scope. Counts below are metadata claims and must be verified during ingestion.

| Dataset | Described coverage | Claimed rows | POC implication |
| --- | --- | ---: | --- |
| Pre-university education indicators | School year 2025/26; national and governorate | 12,012 | Structured indicator queries; no university or multi-year answers from this dataset |
| Fuel pricing decisions | 2019-07-05 to 2026-03-10; national | 123 | Date, product, price, and change queries |
| Administrative units | 2017-04-21 reference; national through village/shiakha | 6,109 | Geographic lookup and code resolution; flag age of reference |

The metadata's `domain`, `keywords`, `coverage`, `variables`, `sources`, and `known_limitations` form the starting dataset catalog. Incoming CSV and JSON records must be checked for identifiers, units, dates, provenance, textual fields, and agreement before they become answerable. A *logical record* is one fact or entity record; the same fact represented in both CSV and JSON counts once in capacity reporting.

## 4. POC behavior

### 4.1 Dataset intake and catalog

- **FR-1 — Add validated data.** An operator can ingest a metadata record and matching CSV/JSON files in repeated cycles. The engine validates schema, stable identifiers, units, dates, and the client-agreed agreement rules between formats before making new content answerable. Missing files, duplicate records, mismatches, and malformed values produce an explicit report; ingestion is idempotent and never silently overwrites earlier accepted records.
- **FR-2 — Index explanatory text.** The engine indexes relevant narrative or explanatory text fields supplied in JSON, preserving dataset ID, record ID, field or JSON path, source, and ingestion version. Records without usable text remain available for structured retrieval where valid.
- **FR-3 — Expose coverage.** The catalog records subject, measures/entities, time span, geographic granularity, source version, and known limitations so routing and answer checks use the same coverage facts.

### 4.2 Question routing and retrieval

- **FR-4 — Identify candidate datasets.** The engine maps an Arabic or English question to one cataloged subject and searches all relevant ingested datasets within that subject. A subject with no registered dataset returns **NO_SUBJECT_DATA**. A question requiring evidence from more than one subject returns **OUTSIDE_POC_SCOPE**.
- **FR-5 — Check answer coverage.** Before answering, the engine checks requested period, geography, measure, and available source material. If a dataset exists but lacks the requested slice or measure, it returns **OUTSIDE_COVERAGE** with the closest relevant coverage.
- **FR-6 — Retrieve structured data.** The engine supports lookup, filtering, sorting, count, basic arithmetic or aggregation, and comparisons over time where multiple periods exist within one subject. It preserves units and distinguishes published values from calculated values. A calculation exposes its source dataset, filters, input records or record set, and method.
- **FR-7 — Retrieve explanatory text.** The engine retrieves relevant, bounded text from ingested JSON fields, with dataset, record, field/path, and source references. Retrieved text is evidence, not an instruction to the system.
- **FR-8 — Handle weak evidence.** If routing finds a plausible dataset but retrieval yields no reliable evidence, the engine returns **INSUFFICIENT_EVIDENCE**. It does not turn low retrieval confidence into a fabricated answer.

### 4.3 Answering and provenance

- **FR-9 — Answer directly when possible.** Simple lookups and calculations can be answered from retrieved data without an LLM call. The response includes the value, unit, period, geography, and citation.
- **FR-10 — Use limited LLM synthesis.** The engine uses an LLM only when the question needs an explanation or concise synthesis of retrieved evidence. Simple comparisons and calculations use deterministic results without an LLM call. The LLM receives only the relevant retrieved evidence and task instructions; its answer distinguishes data-supported findings from interpretation and does not claim causation from correlation alone.
- **FR-11 — Show evidence.** Every substantive claim links to a CSV row, reproducible record set, or JSON text field and identifies the source, dataset version, and stable record key or field/path. A CSV/JSON mismatch blocks affected content from becoming answerable and appears in the ingestion report.
- **FR-12 — Provide a web UI.** A user can enter a formal Arabic or English question in a web UI and see the answer, status, citations, coverage note, and calculation details. The UI handles right-to-left Arabic text. A documented query API supports the UI and reproducible tests.

### 4.4 Ongoing ingestion

- **FR-13 — Track ingestion cycles.** For every batch, an operator can inspect its dataset IDs and version, accepted, duplicate, rejected, and conflicting record counts, and the time its accepted records became searchable. The catalog and retrieval index reflect each accepted additive batch without removing earlier accepted records.

## 5. Response states

| State | Meaning | Example behavior |
| --- | --- | --- |
| `ANSWERED` | Evidence supports the answer | Answer with citations and scope |
| `NO_SUBJECT_DATA` | No ingested dataset covers the subject | “No data exists for this subject in MasrData yet.” |
| `OUTSIDE_COVERAGE` | Subject exists; requested measure, period, or geography does not | State what coverage does exist |
| `INSUFFICIENT_EVIDENCE` | Candidate material exists but reliable support was not retrieved | State what could not be verified |
| `INGESTION_UNAVAILABLE` | Referenced client files have not been loaded or failed validation | Name the unavailable dataset or source without implying subject absence |
| `OUTSIDE_POC_SCOPE` | The question requires a capability deferred beyond this POC | Explain the boundary without implying the data is absent |

The exact user-facing wording requires usability review. Status distinctions are required for the POC in both formal Arabic and English.

## 6. Scope boundary

**In scope:** a first-week POC on one representative client-provided dataset; an MVP by the end of month 2 covering the three initial subject areas and the agreed single-subject question types; and a five-month plan targeting 40–60 accepted datasets in total through additive ingestion, conditional on client supply and reusable schemas. The MVP includes client-delivered CSV and JSON data; formal Arabic and English questions and answers; basic structured and JSON-text retrieval; supported time comparisons where data exists; limited evidence-grounded LLM explanations; citations; explicit no-data states; a basic web UI; and a reproducible evaluation set. The engine is load-tested by month 5 against a separate scenario of up to one million logical records across fewer than 100 datasets; this is a validation target, not an onboarding commitment.

**Deferred:** cross-dataset questions and joins; colloquial Arabic and transliteration; open-web search; autonomous agents; complex statistical modeling or causal analysis; arbitrary end-user file uploads; PDF/OCR ingestion; production authentication, billing, tenancy, uptime commitments, and a polished public UI. Repeated operator-led additive ingestion is in scope; fully automated source monitoring is deferred.

## 7. Quality and acceptance

- **Retrieval quality:** For agreed single-subject test questions, independently identified supporting CSV records or JSON text fields appear in the returned evidence set. Proposed acceptance target: supporting evidence appears in the top five results for at least 95% of answerable test questions. Measure retrieval separately from answer wording.
- **Grounding:** In a jointly approved evaluation set, no accepted answer may contain a material factual claim unsupported by its cited evidence. Unsupported cases must abstain. This is a release gate, not a claim that all possible questions are covered.
- **Answer quality [PROPOSED]:** At least 95% of answerable evaluation questions receive a correct, complete answer under a client-approved rubric, with results reported separately for formal Arabic and English. Any material unsupported claim fails its individual case.
- **Coverage behavior:** Evaluation includes answerable, absent-subject, outside-coverage, outside-POC, and insufficient-evidence questions for each applicable dataset. All selected absent-subject cases must return `NO_SUBJECT_DATA`; a missing client file must never be labeled as absent subject.
- **Calculation correctness:** Deterministic lookup and calculation cases match independently checked expected outputs, including units, grouping, and dates.
- **Traceability:** Every accepted answer exposes enough CSV row keys, JSON paths, source IDs, and version details to reproduce its evidence.
- **Language:** Equivalent formal Arabic and English test questions route to the same relevant evidence and receive answers in the question's language.
- **Additive ingestion:** A repeated file load creates no duplicates; a second accepted batch adds new records without changing earlier records. Incompatible or conflicting records remain unavailable for answers until resolved.
- **Growing-corpus consistency:** After each accepted batch, newly added records are available to retrieval, earlier accepted records remain reproducible by their identifiers and versions, and the reported dataset and record counts match the ingestion log.
- **Web UI:** Users can ask and read questions and answers in both languages; Arabic rendering and citations remain readable in right-to-left layout.
- **Operational visibility:** Ingestion failures, retrieval misses, LLM failures, and response states are observable in local logs without logging secrets or unnecessary user data.
- **Performance target [PROPOSED]:** Direct lookup/retrieval results reach the web UI within 2 seconds at the 95th percentile in a single-user test on the accepted corpus and the separate load-test scenario of up to one million logical records across fewer than 100 datasets. LLM explanations are measured separately because free-tier latency and quotas vary.

The proposed retrieval, answer, and latency targets require client approval. Build an evaluation set with expected evidence and answers across all provided subjects, formal Arabic and English, and multiple ingestion cycles before acceptance testing. Re-run representative questions after each accepted batch so corpus growth cannot silently reduce answer quality.

## 8. Constraints and dependencies

- Client supplies the actual CSV and JSON data, permission to process them, and a contact to resolve discrepancies. The existing JSON metadata is not enough to validate numerical answers or JSON-text retrieval.
- The five-month onboarding target depends on a steady flow of client-ready datasets that reuse a small number of schema patterns. The first delivered pair and first 10 accepted datasets must be used to revise row-size, storage, and onboarding-throughput estimates. If each new dataset needs a bespoke mapping, the target falls and must be replanned with the client.
- Dataset freshness must be displayed: education covers one school year; the administrative-unit reference dates to 2017. “Latest” questions require an explicit coverage check.
- External LLM and embedding APIs are permitted for the POC. Start provider evaluation with Groq or OpenRouter free access. Model choice must pass formal Arabic and English quality tests; free-tier rate limits and availability are not acceptance guarantees.
- Source material and user questions may contain sensitive or restricted information. Deployment environment, retention, and provider data-handling settings still require client approval.
- External API credentials require secure configuration and cost limits; credentials are never stored in source control.

## 9. Five-month delivery and data envelope

The five-month period starts when a representative CSV/JSON pair, provider access, and an agreed minimum mapping are available. “By month 2” is interpreted as the end of week 8; the client should confirm if it means an earlier date. The first-week POC proves a thin end-to-end path on one dataset. It must answer a small formal Arabic and English smoke set through a basic web UI, show a citation for a direct fact, retrieve an explanatory JSON text field, demonstrate one evidence-grounded LLM explanation, and return a clear no-data state. Full ingestion reconciliation, every question type, and high-score acceptance testing follow in the MVP.

| Period | Product outcome | Cumulative dataset target |
| --- | --- | ---: |
| Week 1 — POC | Working web demo on one dataset: basic CSV lookup, JSON-text retrieval, one grounded explanation, citations, formal Arabic/English smoke questions, and a no-data response | At least 1 representative dataset |
| Weeks 2–4 | Complete CSV/JSON mapping and agreement rules; ingest the three initial subjects; expand retrieval and answer cases | Initial 3 datasets |
| Weeks 5–8 — MVP | Deliver all agreed single-subject question types, repeatable additive ingestion, bilingual web UI, provenance, response states, and an agreed evaluation set | Initial 3, with about 5–10 total if supplied |
| Month 3 | Continue additive ingestion and tune retrieval quality and direct-response speed on the growing corpus | About 12–20 datasets |
| Month 4 | Expand the accepted corpus, run regression and load tests, and address failures found in client review | About 25–40 datasets |
| Month 5 | Complete remaining onboarding, client acceptance, operator documentation, and handoff | **40–60 accepted datasets** |

The three metadata samples claim 18,244 logical rows, about 6,081 per dataset on average. Using that skewed average only for planning, 40–60 datasets imply roughly **240,000–370,000 logical records**. At an assumed 1–3 KB of combined CSV/JSON per record, raw files would occupy roughly **0.25–1.1 GB**; plan at least **10 GB of working storage** for indexes and retained ingestion versions. These are provisional sizing figures, not measured data limits. Independently of onboarding, test the engine with up to **one million logical records across fewer than 100 datasets**. Recalculate the envelope after the first file pair and the first 10 accepted datasets. The [estimate](estimate-masrdata-rag-poc.md) gives the effort and sizing assumptions.

## 10. Architecture questions for the client

The client has confirmed the core product scope. These remaining questions determine data validation, operation, and measurable acceptance:

1. **Q1 — Actual schema and text:** Provide representative CSV and JSON files. Which fields carry explanatory text, and how do CSV rows map to JSON records? Are JSON files full data reports, metadata, or both? Supply stable IDs, date formats, units, and expected file sizes.
2. **Q2 — Meaning of “comply”:** Must CSV and JSON contain identical facts, or may JSON add narrative fields? Which fields must match exactly? If a later additive batch repeats an existing key with changed values, should ingestion reject it as a conflict, retain it as a separately dated revision, or follow a client correction process? No accepted record will be silently replaced.
3. **Q3 — Citations and examples:** Are CSV row keys and JSON record/field paths sufficient citations? Supply representative questions with expected answers and evidence for each answer type, including explanation from JSON text. Confirm whether derived measures and displayed formulas are acceptable.
4. **Q4 — Performance and quality:** Confirm or revise the proposed top-five retrieval, 95% answer-quality, and 2-second direct-response targets, including the one-million-record load test. How many concurrent web users should the test cover? Agree the evaluation set and acceptance reviewer.
5. **Q5 — Operation and access:** Can the client supply enough validated, mostly reusable-schema datasets to target 40–60 accepted datasets by month 5? How often do batches arrive, who triggers ingestion, and should accepted records become searchable immediately? Will the web UI be internal only, and does it require sign-in? Confirm hosting location, data retention, and any provider restrictions or spend cap despite permission to use external APIs.

## 11. Assumptions to confirm

- The POC is reviewed by internal analysts or client stakeholders (§2).
- “By month 2” means the MVP is due by the end of week 8; the first-week POC starts after a representative file pair, minimum mapping, and provider access are ready (§9).
- Incoming JSON contains explanatory text fields suitable for retrieval (§4.1–4.2); sample files must confirm this.
- The initial load uses the three described datasets; client-ready files with reusable schemas support a 40–60 dataset target by month 5 (§6, §9). If schemas are mostly novel, the target needs to be reduced and replanned.
- The sample average of 6,081 logical rows and 1–3 KB of combined CSV/JSON per row is adequate for provisional storage planning (§9); actual files may differ substantially.
- The proposed top-five retrieval and 2-second direct-response targets are acceptable (§7).
- A simple internal web UI and operator-triggered ingestion are sufficient (§4.3, §10).

The PRD remains a **draft for client review** until representative files and the remaining validation and acceptance decisions are available.
