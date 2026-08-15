# Testing Guide — Query Frontend

This guide covers how to run and extend tests for the query frontend changes.

## Running tests

```bash
go vet ./pkg/queryfrontend/...
go test ./pkg/queryfrontend/... -count=1
```

For race detection on the cache and invalidation paths:

```bash
go test ./pkg/queryfrontend/... -race -run TestCache
```

## Test matrix

| Area | What to assert |
|---|---|
| Cache key | Determinism for a fixed (tenant, query, epoch) triple |
| Cache key | A changed epoch yields a different key |
| Invalidation | In-flight query is not committed under a stale epoch |
| Concurrency | No data race between key generation and epoch reads |

## Writing tests

- Keep fixtures minimal; a single tenant/query pair is usually enough.
- Simulate a rebalance by bumping the epoch in the test and asserting the
  observable behavior (new key, no stale commit).
- Use `-race` in CI for the cache path since it is concurrent by design.
- Avoid wall-clock time in assertions; the invalidation logic must be driven by
  explicit epoch transitions, not timers.
