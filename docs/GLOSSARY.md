# Glossary

Definitions of terms used throughout the query-frontend documentation.

## Cache key

The string used to look up a cached query result. Composed of the tenant
identifier, the query fingerprint, and the active routing epoch.

## Routing epoch

A monotonically increasing identifier for a tenant's routing assignment. The
epoch changes whenever the tenant's node assignment changes (for example
during a Shuffle-Sharding rebalance).

## Shuffle-Sharding

A technique that limits the blast radius of a node failure by assigning each
tenant to a small subset of nodes rather than all nodes.

## Rebalance

A change to tenant-to-node assignments, triggered by node churn or load
rebalancing. A rebalance bumps the routing epoch for affected tenants.

## Mid-flight invalidation

A signal that prevents a query that started before a rebalance from being
committed to the cache under the stale epoch.

## Key collision

Two different (tenant, query) combinations producing the same cache key,
which would return a wrong result. Prevented by including the routing epoch
in the key.

## Partial result

A result computed against a stale topology snapshot. Prevented by the
mid-flight invalidation mechanism.

## Hot path

A code path executed on every request. Hot paths must minimize allocations
and latency.

## Allocation-free

Producing no heap allocations, typically by using pre-sized buffers and
`strings.Builder`.

## Epoch churn

Frequent, repeated changes to the routing epoch, which depresses cache
effectiveness by constantly invalidating entries.
