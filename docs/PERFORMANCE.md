# Performance Guide — Query Frontend

This guide describes the performance characteristics of the query frontend
and how to keep the hot paths fast.

## Hot paths

The query frontend has three hot paths:

1. **Key generation** — runs on every query. Must be allocation-free.
2. **Cache lookup** — a map lookup on every query. Must be O(1).
3. **Result assembly** — runs when a result is built from cached parts.

## Key generation

Key generation is the most sensitive path. Rules:

- Use `strings.Builder` with a buffer pre-sized to the expected key length.
- Append the tenant id, query fingerprint, and routing epoch in a fixed order.
- Never call `fmt.Sprintf` or string concatenation in a loop.
- Never include wall-clock time or random values.

A regression that adds allocations here shows up immediately as GC pressure
and elevated CPU under load.

## Cache lookup

- Keep the cache as a hash map keyed by the cache key string.
- Avoid resizing during steady state; pre-size if the key space is known.
- Eviction should be O(1) or O(log n) and must not block query serving.

## Avoiding stale work

- During a rebalance, in-flight queries must not be committed under a stale
  epoch; the invalidation signal avoids wasted work and wrong results.
- Coalesce duplicate in-flight queries for the same key where possible.

## Benchmarking

Benchmarks must cover:

- Key generation throughput and allocation count (assert zero allocations).
- Cache lookup with a realistic number of entries.
- End-to-end query latency under a rebalance.

Run benchmarks with `-benchmem` to verify the allocation-free guarantee:

```bash
go test ./pkg/queryfrontend/... -bench . -benchmem -run '^$'
```

## Regression checklist

Before merging a change that touches these paths, confirm:

- [ ] Key generation still allocates zero bytes per call.
- [ ] Cache lookup remains O(1).
- [ ] End-to-end p99 latency is unchanged outside rebalances.
