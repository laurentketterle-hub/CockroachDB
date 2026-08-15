# Operations Runbook

This runbook covers day-to-day operations for the query frontend.

## Health signals

Monitor the following:

- **Query latency** (p50/p99) — regressions point to cache or routing issues.
- **Cache hit rate** — a healthy deployment has a stable hit rate between
  rebalances.
- **Key-generation allocation rate** — the hot path should be allocation-free;
  spikes indicate a regression in the key builder.
- **Epoch churn** — frequent epoch changes indicate unstable routing and will
  depress cache effectiveness.

## Rebalance procedure

A Shuffle-Sharding rebalance changes tenant-to-node assignments. Expected
behavior:

1. The routing epoch bumps.
2. Cache keys for moved tenants change.
3. A transient cache-miss burst occurs while new keys populate.
4. In-flight queries are not committed under the stale epoch.

No operator action is required for the cache; it self-heals as new keys
populate.

## Alerting

| Alert | Threshold | Response |
|---|---|---|
| Zero cache hits | 5 minutes | Investigate epoch roll or config change |
| p99 latency spike | 2x baseline | Check node assignment and cache health |
| High key-gen allocations | sustained | Revert recent key-builder change |

## Incident response

1. Check whether a rebalance or configuration change occurred recently.
2. Confirm the routing epoch is being read atomically.
3. Verify the invalidation signal is propagated during rebalances.
4. Roll back the last change if a code change correlates with the incident.

## Capacity planning

- Cache memory grows with the number of distinct (tenant, query) keys.
- Rebalances temporarily double the key space; size capacity for that
  transient.
