---
title: Fuzzy Name Matching with a BiLSTM Person/Entity Classifier
summary: Production name-matching pipeline that improved match accuracy by 47% and reduced false positives by 25%, with a learned person-vs-entity classifier driving most of the late-stage gains.
role: Software Development Engineer
timeframe: "2023 – Present"
stack:
  - Python
  - PyTorch
  - PostgreSQL
  - Flask
  - Celery
order: 1
impact:
  - 47% improvement in match accuracy across the production matcher
  - 25% reduction in false positives on customer-tagged regression sets
  - 22% accuracy gain from the BiLSTM person/entity classifier alone (Malaysian naming patterns)
---

> **Note:** This page describes the approach and measured outcomes only. It does not include
> proprietary data, customer-specific heuristics, or internal architecture details.

## Problem

Bank-statement processing relies on matching names — account holders, payees, beneficiaries —
to resolve identity across millions of records. Traditional fuzzy matching (Levenshtein,
phonetic algorithms, token-set similarity) breaks down on multi-script, multi-region naming
conventions, where the *structure* of a name carries information that string similarity alone
cannot recover.

The most painful failure mode was conflating personal names with business/entity names that
shared common tokens, especially in markets like Malaysia where naming conventions diverge
sharply from typical Latin-script assumptions.

## Approach

I rebuilt the matching pipeline as a layered system:

1. **Normalization layer** — script-aware tokenization, transliteration handling, and
   regional stop-token removal.
2. **Candidate generation** — fast blocking via indexed n-gram signatures to keep the
   downstream classifier inputs bounded.
3. **Similarity features** — multiple distance signals combined into a feature vector rather
   than a single threshold.
4. **Person/entity classifier** — a BiLSTM over character-level tokens with token alignment,
   trained on labelled examples curated with input from customers, QA, and product.

## Key decisions

- **Why BiLSTM over a simpler classifier:** sequence context matters for entity detection —
  transformer-style attention was overkill for the latency budget and training data size,
  while bag-of-tokens classifiers lost the positional signal that distinguishes
  "Sons & Co" appearing mid-name from being the head of an entity.
- **Why not just throw an LLM at it:** sub-200ms latency budget, on-prem deployments with
  no outbound network, and the need to retrain on customer-tagged corrections quickly.
- **Customer collaboration was the highest-leverage input.** The single biggest accuracy
  lift came from working directly with customers, sales, product, and QA to surface naming
  patterns that didn't exist in our training distribution.

## Results

The 47% headline match-accuracy gain was the cumulative effect of all the layers; the
classifier itself contributed 22% of the lift. False positives — the metric customers
actually feel — dropped 25%.

## What I'd do differently

I'd invest earlier in a labelling/feedback loop so that customer corrections flowed back
into the training set without manual curation. Most of the time spent on the BiLSTM
component went into data work, not modelling.
