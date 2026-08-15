# Query-Frontend Cache Key Design

This document describes the cache key generation and invalidation strategy for
the query frontend, and the reasoning behind the topology-aware design.

## Problem

The query frontend caches query results keyed by the query fingerprint. Under
Shuffle-Sharding rebalances, the set of tenants mapped to a given node changes.
A cache key that ignores routing topology risks:

- **Key collisions** — two different tenant/query combinations producing the
  same key and returning the wrong result.
- **Partial results** — a rebalance mid-flight returning a result computed
  against a stale topology.

## Design

### Topology-aware keys

The key generator incorporates the active routing epoch of the tenant into the
cache key. Because the epoch changes whenever the tenant's node assignment
changes, a rebalance produces new keys and invalidates stale entries naturally.

Key construction uses `strings.Builder` with pre-sized buffers to keep key
generation allocation-free on the hot path.

### Mid-flight invalidation

In addition to epoch-scoped keys, an invalidation signal is recorded so that
any query in flight during a rebalance is not committed to the cache under the
old epoch. This prevents stale partial results from being served after the
topology has moved.

## Correctness constraints

- **Deterministic keys.** The same tenant, query, and epoch must always produce
  the same key. No wall-clock time or random values may enter the key.
- **Allocation-free hot path.** Key generation runs on every query; avoid
  allocations that would add GC pressure under load.
- **Atomic epoch reads.** The epoch must be read atomically so the key and the
  topology snapshot are always consistent.

## Testing

- Unit tests assert that a topology change (epoch bump) produces a different
  key for the same query, and that a stable topology produces a stable key.
- Race tests cover the mid-flight invalidation path during simulated
  rebalances.
