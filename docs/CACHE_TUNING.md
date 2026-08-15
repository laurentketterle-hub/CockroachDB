# Cache Tuning Guide

This guide describes how to tune the query-frontend result cache for
production workloads.

## Key dimensions

The cache has two primary tuning dimensions:

1. **Entry time-to-live (TTL)** — how long a cached result is considered
   fresh.
2. **Capacity** — how many entries (and how much memory) the cache holds.

## TTL guidance

- **Short TTLs** (seconds to a minute) suit volatile data where a stale result
  is costly.
- **Long TTLs** (minutes to an hour) suit stable data and reduce upstream load.

Choose the TTL based on how often the underlying data changes. If data changes
more often than the TTL, users will observe stale results.

## Capacity guidance

Cache memory grows with the number of distinct `(tenant, query)` keys. A
Shuffle-Sharding rebalance temporarily doubles the key space for moved
tenants. Size capacity to absorb that transient rather than for steady state.

## Epoch churn

The cache key includes the tenant's routing epoch. Frequent epoch changes:

- Depress the cache hit rate (new keys on every change).
- Increase key-generation work.

If epoch churn is high, investigate the routing stability before tuning the
cache. Reducing unnecessary rebalances is more effective than enlarging the
cache.

## Observability

Track:

- **Hit rate** — a stable rate between rebalances is healthy.
- **Miss burst** — expected immediately after a rebalance.
- **Entry churn** — high churn outside rebalances indicates TTL too short or
  unstable key inputs.

## Hot-path allocation

Key generation runs on every query. Keep it allocation-free:

- Use `strings.Builder` with a pre-sized buffer.
- Avoid `fmt.Sprintf` in the key builder.
- Never include wall-clock time or random values in the key.

## Common misconfigurations

| Misconfiguration | Symptom | Fix |
|---|---|---|
| TTL too long | Stale results served | Shorten TTL |
| TTL too short | Low hit rate, high upstream load | Lengthen TTL |
| Capacity too small | Excessive evictions | Increase capacity |
| Key missing epoch | Wrong results after rebalance | Add epoch to key |
