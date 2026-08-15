# Frequently Asked Questions — Query Frontend

## Caching

### Why did my cached result change after a rebalance?

The cache key includes the tenant's routing epoch. A rebalance bumps the
epoch, which produces new keys and invalidates the previous entries. This is
intentional: it prevents stale results from the old topology.

### How do I know a rebalance happened?

Watch for a transient cache-miss burst. The routing epoch change is the
expected cause.

### Can I cache without the topology-aware key?

No. A key that ignores routing topology risks collisions and partial results
under Shuffle-Sharding rebalances.

## Key generation

### What goes into the cache key?

The tenant identifier, the query fingerprint, and the active routing epoch.
Nothing else — no wall-clock time and no random values.

### Why must key generation be allocation-free?

Key generation runs on every query. Allocations on this hot path add GC
pressure and CPU cost under load.

## Correctness

### What happens to an in-flight query during a rebalance?

An in-flight query is not committed to the cache under the stale epoch. The
mid-flight invalidation signal prevents it from being served after the
topology has moved.

### Are keys deterministic?

Yes. The same tenant, query, and epoch always produce the same key.

## Operations

### How do I tune the TTL?

Set the TTL based on how often the underlying data changes. Volatile data
needs a short TTL; stable data can use a long TTL.

### How much cache capacity do I need?

Size for the steady-state key space plus the transient doubling during
rebalances.
