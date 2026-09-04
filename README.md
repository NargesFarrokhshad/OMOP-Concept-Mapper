# OMOP Concept Mapper

A human-in-the-loop, embedding-based web assistant for mapping local clinical
source values to OMOP Standard Concepts — built to reduce repetitive Athena
searches and speed up first-pass mapping during OMOP CDM implementation,
while keeping a human in control of every final decision.

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

## Note

This repository contains the front-end interface only. It talks to a
separate backend NLP/retrieval service (not included here) via a REST API.
