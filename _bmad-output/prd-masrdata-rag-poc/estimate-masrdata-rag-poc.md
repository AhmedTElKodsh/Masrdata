# MasrData RAG POC — five-month delivery and data estimate

**Prepared:** 2026-10-11
**Status:** preliminary five-month plan, measured from receipt of representative CSV/JSON files and agreement on their mapping. Revisit after the files are inspected and the remaining decisions in the [PRD](prd-masrdata-rag-poc.md) are answered.

## Basis

This estimates a web-based POC using the three described datasets as its initial corpus (18,244 rows claimed in metadata), then continues additive CSV/JSON onboarding through month 5. It answers single-subject questions in formal Arabic and English through a basic web UI. It includes structured retrieval, retrieval from explanatory JSON text where supplied, limited LLM synthesis, citations, explicit no-data states, and repeated evaluation. The planning target by the end of month 5 is **40–60 accepted datasets total**, assuming most new files follow reusable schemas and arrive ready for validation. This is an onboarding target, not a hard database limit or a guarantee before the files are seen.

The plan assumes one experienced full-stack/AI engineer with intermittent client product/data review. A person-day is eight focused hours. Five months is approximately 20 weeks or 100 nominal working days; the effort range below leaves varying room for client review and delivery risk.

| Work package | Person-days | Deliverable |
| --- | ---: | --- |
| Inspect delivered CSV/JSON and agree evaluation questions | 3–4 | Schema mapping and expected evidence set |
| Build additive ingestion, reconciliation, and provenance | 6–8 | Idempotent loads, conflict report, indexed accepted records |
| Build catalog routing and coverage states | 2–3 | Subject/measure selection and honest abstention |
| Build structured retrieval | 3–4 | Filters, calculations, row-set provenance |
| Build explanatory JSON-text retrieval | 2–3 | Text search and record/field citations |
| Add limited LLM synthesis and answer controls | 2–3 | Grounded explanations and citation checks |
| Provide query API and basic bilingual web UI | 3–4 | End-to-end POC with right-to-left Arabic support |
| Evaluate, fix failures, prepare MVP review | 5–6 | Retrieval/answer benchmarks, ingestion-cycle checks, operator notes |
| **Subtotal** | **26–35** | |
| Contingency for ordinary source and integration variance | **3–5** | |
| **MVP build through week 8** | **29–40 person-days** | Agreed single-subject question types and bilingual web UI |

The first-week POC is a 4–5 person-day subset of the MVP build, not additional effort. It demonstrates one representative CSV/JSON pair, direct lookup, JSON-text retrieval, one grounded LLM explanation, citations, formal Arabic/English smoke questions, a no-data response, and a basic web UI. The full reconciliation, question-type coverage, and quality gates are due with the MVP by the end of week 8. This schedule depends on a clean representative pair, minimum mapping decisions, and provider access being ready when the five-month clock starts.

After the MVP, months 3–5 concentrate on repeated ingestion, regression work, and acceptance on a growing corpus:

| Extension work during months 3–5 | Person-days | Deliverable |
| --- | ---: | --- |
| Onboard and validate additional client-ready CSV/JSON datasets | 18–25 | Accepted batches, mappings, mismatch reports, versioned provenance |
| Tune retrieval and direct-response performance; run regression and load checks as data grows | 8–11 | Measured retrieval quality and latency across successive loads |
| Client acceptance rounds and answer-quality fixes | 5–7 | Reviewed bilingual results and resolved priority failures |
| Operator documentation and handoff | 3–4 | Repeatable ingestion procedure and technical handoff |
| **Additional effort** | **34–47** | |
| **Five-month effort range** | **63–87 person-days** | **20 calendar weeks for one engineer** |

The upper end leaves roughly 13 nominal working days for client reviews, waiting, and ordinary schedule variance; the lower end leaves more slack. The week-8 MVP remains the tightest milestone: its 40 person-day upper bound consumes the nominal first eight weeks. Client feedback delays, unusually varied schemas, or large files would reduce the dataset target or require more time or staffing. The work packages may overlap; dataset ingestion and evaluation continue throughout the five months.

## Estimated data handled in five months

The metadata claims 12,012 education rows, 6,109 administrative-unit rows, and 123 fuel rows: **18,244 logical rows across three datasets**, or about **6,081 rows per dataset on average**. That average is highly skewed and is only a sizing reference until actual files arrive. A logical row means one fact/entity record; the same fact appearing in CSV and JSON is counted once.

| Planning measure | Five-month estimate | Basis and limit |
| --- | --- | --- |
| Accepted datasets | **40–60 total**, including the initial three | Assumes reusable schemas, stable IDs, and prompt client delivery; fewer if each dataset needs bespoke mapping |
| Logical records | **About 240,000–370,000** | 40–60 datasets × roughly 6,081 rows; actual subject mix may differ substantially |
| Raw CSV + JSON storage | **About 0.25–1.1 GB** | Assumes 1–3 KB of combined CSV/JSON bytes per logical record; measure this from sample files |
| Storage including indexes and retained ingestion versions | **Plan for at least 10 GB** | Working allowance, not a measured requirement; text length, embeddings, and version retention can raise it |
| Engine load-test scenario | **Up to 1 million logical records across fewer than 100 datasets** | Performance validation target; it does not mean all those datasets will be onboarded by month 5 |

The first delivered CSV/JSON pair should be used to measure bytes per record, explanatory-text length, duplicate rate, and ingestion time. Recalculate the dataset and storage envelope after that audit and again after the first 10 accepted datasets.

## Assumptions behind the range

- The client supplies representative CSV and JSON files promptly. Explanatory text appears in JSON fields and can be mapped to stable record IDs; no PDF/OCR parsing is included.
- CSV and JSON agreement rules, duplicate-key behavior, and additive version semantics are settled during the file audit. The estimate includes repeatable operator-triggered ingestion, not unattended source monitoring.
- The POC is designed and load-tested for fewer than 100 datasets and one or a few reviewers. The five-month onboarding target is 40–60 datasets; record volume and concurrency remain to be confirmed from actual files.
- Most additional datasets reuse a small number of CSV/JSON schema patterns and arrive with stable IDs and client-approved agreement rules. If each dataset has a novel schema, the five-month onboarding target falls to roughly 15–30 datasets pending a new sample-based estimate.
- Groq or OpenRouter free access is used for initial LLM trials, with provider/model selection based on formal Arabic and English evaluation. Embedding needs and provider are validated during the retrieval baseline. Free-tier quotas can constrain testing; API charges and hosting costs are **not** included.
- Formal Arabic and English queries and answers are included. Colloquial Arabic and transliteration tuning are excluded.
- Cross-dataset questions and joins, production authentication, a polished public UI, compliance certification, and production operations are excluded.
- Client reviewers can answer architecture questions and review the evaluation set within two working days per checkpoint. Waiting time beyond that extends calendar duration.

As checked on 2026-10-11, [Groq documents model-specific free-plan rate limits](https://console.groq.com/docs/rate-limits), while [OpenRouter lists 50 requests/day on its Free plan](https://openrouter.ai/pricing/) and exposes [embeddings as a separate API](https://openrouter.ai/docs/api/api-reference/embeddings/create-embeddings). These are trial options, not fixed project capacity or a promise that embeddings will be free. Verify the actual account limits and model prices before choosing a provider or scheduling the full evaluation run.

## Five-month schedule checkpoints

| Period | Checkpoint | Exit evidence |
| --- | --- | --- |
| Week 1 — POC | Narrow end-to-end demo | One representative dataset; CSV lookup, JSON text, one grounded explanation, citations, bilingual smoke questions, and no-data state |
| Weeks 2–4 | Complete data mapping and retrieval foundation | Initial three datasets load repeatably; agreement rules, direct lookups, and citations work |
| Weeks 5–8 — MVP | Complete agreed single-subject capabilities | Bilingual web UI, lookup, ranking/aggregation, supported time comparisons, JSON-text explanations, response states, additive ingestion, and evaluation set; about 5–10 datasets total if supplied |
| Month 3 (weeks 9–12) | Additive ingestion and tuning | About 12–20 datasets accepted; retrieval and direct-response targets measured on growing data |
| Month 4 (weeks 13–16) | Corpus growth and regression | About 25–40 datasets accepted; load test and client-review findings addressed |
| Month 5 (weeks 17–20) | Acceptance and handoff | Target 40–60 datasets accepted; final evaluation results, ingestion procedure, and handoff complete |

## Estimate changes to watch

| Client answer or discovered condition | Likely impact on planning range |
| --- | --- |
| CSV/JSON have no stable join keys or disagree in many fields | Re-estimate after reconciliation policy and representative error rate are known |
| JSON contains little or no explanatory text | Revise retrieval/LLM scope after examining examples |
| Single-subject time-series questions beyond available periods | Re-estimate after additional data and examples are reviewed |
| Cross-dataset questions or joins | Future phase with a separate scope and estimate |
| Authentication, multi-user deployment, or operational SLA | Separate delivery phase and estimate |
| Free-tier provider limits block evaluation or demo volume | Allow paid API budget or extend schedule; confirm with current account quotas |
| Additional datasets with new schemas | Estimate per dataset after a sample and mapping rules are available |
| Files average much larger than the three metadata examples imply | Recalculate storage and performance envelope from actual record sizes and text lengths |

**Confidence:** moderate for the MVP build, low for the 40–60 dataset and storage projections until representative files and the first 10 accepted datasets are measured. The five-month period is a delivery plan with a capacity target; later product expansion needs its own scope and estimate.
