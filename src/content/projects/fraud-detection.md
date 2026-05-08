---
title: Fraud Detection on Bank Transactions with Polars
summary: Rebuilt the fraud-detection engine on top of Polars with spend-category and transaction-mode signals, lifting detection accuracy by 28% on backtested data.
role: Software Development Engineer
timeframe: "2024 – 2025"
stack:
  - Python
  - Polars
  - Pandas
  - PostgreSQL
order: 3
impact:
  - 28% improvement in fraud detection accuracy on backtested customer data
  - Faster cold-start on large statements via Polars lazy execution
  - "New signals: spend-category and transaction-mode intelligence"
---

> **Note:** This page describes the modelling and engineering approach. It does not
> include customer data, internal heuristics, or any signal that would help an actor
> evade detection.

## Problem

The existing fraud-detection engine flagged anomalies on bank-transaction streams, but it
operated mostly on transaction-level features: amount distributions, frequency, simple
counterparty rules. That missed two classes of behaviour:

- **Spend-category context.** A Rs. 50,000 transaction is unremarkable for some
  category profiles and a strong outlier for others.
- **Transaction-mode patterns.** UPI vs. NEFT vs. card vs. cash deposit each carry
  different baseline expectations; conflating them washed out signal.

Adding these signals naively in Pandas was prohibitively slow on the corpora we needed
to backtest against.

## Approach

1. **Switched the hot path to Polars.** Lazy evaluation and columnar execution let us
   express the new aggregates over months of transaction history without holding the
   full set in memory.
2. **Added category and mode features.** Per-account baselines computed over rolling
   windows, with category-conditional and mode-conditional anomaly scores merged into
   the existing detector.
3. **Backtested against tagged customer cases.** Held out a labelled set of
   known-fraud and known-clean statements and tuned thresholds against precision /
   recall on that set, not on synthetic data.

## Key decisions

- **Polars over Spark.** The data fit on a single beefy node, and the operational
  cost of a Spark cluster wasn't justified.
- **Augment, don't replace.** The new signals plug into the existing detector rather
  than supplanting it. That kept the model auditable and let us A/B-test the new
  signals' contribution cleanly.
- **No public detail on which patterns we flag.** Accuracy gains are reported in the
  aggregate; specific rules and thresholds stay internal.

## Results

The 28% accuracy improvement is on backtested labelled cases — precision and recall
both moved, with the bigger lift on recall (catching genuine fraud the previous
engine missed). Pipeline runtime stayed within the previous SLO despite the extra
features, thanks to the Polars rewrite.

## What I'd do differently

The labelled set for backtesting was smaller than I'd have liked. A more rigorous
labelling effort up front would have let us validate finer-grained thresholds with
more confidence.
