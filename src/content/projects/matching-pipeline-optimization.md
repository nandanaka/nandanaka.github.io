---
title: Cutting Matching-Pipeline Latency and Memory by 50%
summary: A targeted optimization pass on the production matching pipeline that halved both latency and peak memory, lifting throughput on the same infrastructure without algorithmic changes that would affect accuracy.
role: Software Development Engineer
timeframe: "2024"
stack:
  - Python
  - Polars
  - PostgreSQL
  - Flask
  - Celery
order: 2
impact:
  - 50% reduction in end-to-end pipeline latency
  - 50% reduction in peak memory per worker
  - Higher throughput on the same hardware — no accuracy regression
---

> **Note:** This page describes profiling methodology and high-level findings, not internal
> code paths or proprietary heuristics.

## Problem

The matching pipeline that powers fuzzy name resolution had grown organically. Throughput
on the existing infrastructure was the bottleneck for downstream products, and adding more
workers wasn't viable — peak memory per worker was already pushing host limits during
high-volume runs.

The hard constraint: any optimization had to be byte-equivalent on the matching results.
We could not change the answer the pipeline produced, only how it produced it.

## Approach

The work split into three phases, each gated by profiling rather than guesses.

1. **Profile first.** Used py-spy and memray to find where time and memory actually went.
   The intuitive bottleneck and the real one were different — typical of mature systems.
2. **Reduce data movement.** A surprising fraction of CPU was spent on dataframe copies
   and dtype churn. Switching the hot path to lazy/streaming Polars operations and
   carefully managing column dtypes recovered both time and memory at once.
3. **Cache structure, not values.** Several intermediate structures were rebuilt per
   record when they could be built once per batch. Hoisting these out of the inner loop
   was the single largest latency win.

## Key decisions

- **No accuracy-affecting changes were on the table.** That ruled out approximate-match
  shortcuts that would have been faster but would have shifted the matching distribution.
- **I avoided rewriting in a faster language.** A Cython/Rust pass was tempting, but the
  data showed the bottlenecks were in algorithmic shape, not interpreter overhead.
  Profiling-driven Python changes captured the win with far less risk.
- **Memory and latency were the same fix.** The big-win refactors reduced both
  simultaneously — most "memory bloat" was actually "throwaway dataframes the pipeline
  was rebuilding repeatedly."

## Results

End-to-end latency and peak memory both dropped roughly 50%. With memory headroom
recovered, the same hosts now run more workers concurrently, lifting throughput further
than the latency number alone suggests.

## What I'd do differently

Run the profiler in CI on a fixed corpus, so regressions get caught before they ship.
The next person doing this kind of work shouldn't have to rediscover the bottlenecks
from scratch.
