# OMOP Concept Mapper

A human-in-the-loop, embedding-based web assistant for mapping local clinical
source values to OMOP Standard Concepts — built to reduce repetitive Athena
searches and speed up first-pass mapping during OMOP CDM implementation,
while keeping a human in control of every final decision.
<img width="1299" height="652" alt="image" src="https://github.com/user-attachments/assets/35348ab0-08d2-4d11-9b4f-2ce90de9742a" />


## What it does

- Accepts a CSV of local source values and lets you pick the column to map.
- Configures mapping parameters: input language, clinical context, max
  candidates, batch size, optional datapoint-set filtering, similarity
  threshold.
- For each source value, retrieves ranked candidate OMOP concepts via a
  biomedical neural encoder (SapBERT-based) and nearest-neighbour search over
  a curated concept knowledge base derived from UMLS, enriched with OMOP
  Athena mappings.
- Buckets each row into one of three review states based on the score
  distribution:
  - **Auto-mapped** — top candidate clears the threshold with a clear margin.
  - **Tie-break required** — two or more candidates are within a narrow
    margin, so reviewer judgement is needed to disambiguate.
  - **Manual review** — low-confidence retrieval, full search needed.
- Lets reviewers validate, reject, or manually override any mapping before
  exporting a reviewable source-to-concept table (concept ID, domain,
  vocabulary, similarity score, review status) for use directly in ETL.

## Why

Vocabulary mapping is one of the most labour-intensive steps in OMOP CDM
implementations, and fully automated mapping isn't realistic or desirable —
expert oversight matters, especially in multilingual and locally coded data.
This tool aims for the middle ground: automate candidate discovery, keep
humans deciding.

## How it works

1. **Input** — Upload a CSV of local source values and select the column to map.
2. **Configure** — Set input language, clinical context, optional datapoint-set
   filter (restrict candidates to a known set of clinically relevant concepts),
   max candidates per term, batch size, and similarity threshold.
3. **Retrieve** — Each term is embedded with a biomedical entity-linking model
   (SapBERT-based) and matched against a curated concept knowledge base — built
   from UMLS and enriched with OMOP Athena mappings — via nearest-neighbour
   search. Requests are processed in batches, and results can optionally be
   filtered to a specific OMOP domain (e.g. only `Drug` or `Measurement`
   candidates).
4. **Triage** — Each row is scored and bucketed into one of three states:
   auto-mapped, tie-break required, or manual review (see below).
5. **Review** — The reviewer inspects the top candidate and alternatives,
   and validates, rejects, or manually assigns a different OMOP concept.
6. **Learn** — When a reviewer overrides a suggestion, that (source value,
   concept) pair is written back into the knowledge base as a new anchor for
   the corresponding concept. Later batches — even in different projects —
   benefit from that correction automatically, so previously resolved
   abbreviations, lexical variants, and multilingual labels don't need to be
   re-reviewed from scratch.
7. **Export** — A reviewable mapping table (concept ID, domain, vocabulary,
   similarity score, review status) is exported for direct use in ETL.
   <img width="1299" height="652" alt="image" src="https://github.com/user-attachments/assets/abe8e3c2-1e56-4b8f-a2de-ec4c92ba7436" />


## Note

This repository contains the front-end interface only. It talks to a
separate backend NLP/retrieval service (not included here) via a REST API.
