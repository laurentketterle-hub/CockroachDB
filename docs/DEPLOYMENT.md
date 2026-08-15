# Query Frontend Deployment Guide

This document describes how to build, configure, and operate the query
frontend.

## Build

```bash
go build ./pkg/queryfrontend/...
```

## Configuration

The query frontend reads its configuration from the standard configuration
surface. Relevant settings:

| Setting | Purpose |
|---|---|
| Tenant routing epoch source | Where the active routing epoch is read from |
| Cache enablement | Whether result caching is on |
| Cache entry TTL | Default time-to-live for cached results |

## Caching behavior

Results are cached keyed by a topology-aware cache key. The key incorporates
the tenant's routing epoch so that Shuffle-Sharding rebalances naturally
invalidate stale entries. An in-flight invalidation signal prevents partial
results from being committed under a stale epoch.

## Operations

- Watch cache hit rate and entry churn. A sudden drop in hit rate usually
  indicates a rebalance or an epoch roll.
- During a rebalance, expect a transient increase in cache misses while new
  keys are populated.
- Monitor memory used by the cache and adjust capacity if needed.

## Scaling

The query frontend scales horizontally. Cache keys are deterministic and
epoch-scoped, so instances remain consistent even when the routing topology
changes.

## Rollback

Rollback is a redeploy of the previous binary. Because the cache key scheme
includes the epoch, a rollback that changes key generation will naturally
separate entries from the new scheme.

## Common issues

| Symptom | Cause | Action |
|---|---|---|
| Wrong results served | Key collision from ignored topology | Verify epoch is in the key |
| Partial results during rebalance | Missing mid-flight invalidation | Verify invalidation signal is set |
| High key-generation CPU | Allocation-heavy key builder | Confirm `strings.Builder` path |
