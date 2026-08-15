# Troubleshooting Guide

This guide helps diagnose and resolve common query-frontend issues.

## Symptoms and remedies

### Wrong query result served

The cache returned a result for a different tenant or query. This indicates a
cache-key collision.

**Cause:** the routing epoch is missing from the cache key.

**Fix:** verify the key generator includes the tenant's active routing epoch,
and that the epoch is read atomically.

### Partial result after a rebalance

A query started before a rebalance returned a partial result afterwards.

**Cause:** the mid-flight invalidation signal was not propagated.

**Fix:** verify the invalidation signal is set for every query that spans a
rebalance boundary.

### High CPU with no traffic increase

Key generation is allocating on the hot path.

**Cause:** the key builder regressed to `fmt.Sprintf` or repeated string
concatenation.

**Fix:** restore the `strings.Builder` path and confirm zero allocations with
`go test -bench . -benchmem`.

### Cache hit rate near zero

The cache is not serving hits.

**Cause:** the routing epoch is changing frequently, or the TTL is far too
short.

**Fix:** investigate routing stability first, then adjust the TTL.

### Non-deterministic keys

The same query produces different keys across runs.

**Cause:** wall-clock time or a random value entered the key.

**Fix:** remove any time or randomness from the key; keys must depend only on
the tenant, query fingerprint, and epoch.

## Diagnostic commands

```bash
go vet ./pkg/queryfrontend/...
go test ./pkg/queryfrontend/... -count=1
go test ./pkg/queryfrontend/... -race -run TestCache
go test ./pkg/queryfrontend/... -bench . -benchmem -run '^$'
```

## Escalation

If a symptom cannot be explained by the causes above, capture the following
and escalate:

- The cache key for a reproducing query (tenant, fingerprint, epoch).
- The routing epoch timeline around the incident.
- Benchmark output showing allocation counts.
