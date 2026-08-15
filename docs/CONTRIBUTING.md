# Contributing Guide

This guide explains the workflow for contributing to the query frontend.

## Getting started

```bash
go vet ./pkg/queryfrontend/...
go test ./pkg/queryfrontend/... -count=1
```

## Workflow

1. Open an issue describing the bug or feature.
2. Create a branch from `main`.
3. Make the change, keeping the cache-key hot path allocation-free.
4. Add or update tests (unit + race).
5. Run `go vet` and the full test suite.
6. Open a pull request referencing the issue.

## Quality bar

A change must:

- Preserve key determinism for a fixed (tenant, query, epoch) triple.
- Keep the hot path free of wall-clock time and random values.
- Include tests for the observable behavior it changes.
- Pass `go test -race` for any concurrency-sensitive code.

## Cache key invariants

These invariants are load-bearing and must not be broken:

1. Same tenant + query + epoch => same key.
2. Different epoch => different key.
3. No allocation in the key builder beyond the pre-sized buffer.
4. No wall-clock or random input in the key.

## Commit conventions

- Sign off every commit (`git commit -s`).
- Use a `fix:` or `feat:` prefix.
- Reference the issue in the PR description.

## Review expectations

Reviewers verify the cache-key invariants, check for race conditions, and
confirm test coverage of the changed behavior.
