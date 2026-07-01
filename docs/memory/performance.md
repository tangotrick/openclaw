# Memory Performance Tuning

OpenClaw's memory subsystem uses SQLite-vec for vector search. For databases larger than 100MB, cold disk reads can dominate search latency (10-12s on first call, ~600ms warm).

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MEMORY_SEARCH_TOOL_TIMEOUT_MS` | `60000` | Max wait for a memory_search tool call. Was hardcoded to `15000`. |
| `MEMORY_DB_CACHE_KB` | `200000` | SQLite `cache_size` (negative = KB). ~200MB OS page cache reservation. |
| `MEMORY_DB_MMAP_BYTES` | `268435456` | SQLite `mmap_size`. 256MB memory-mapped I/O window. |

## Recommended values for large DBs (>= 200MB)

```bash
export MEMORY_SEARCH_TOOL_TIMEOUT_MS=60000     # was 15e3
export MEMORY_DB_CACHE_KB=200000               # ~200MB cache
export MEMORY_DB_MMAP_BYTES=268435456          # 256MB mmap
```

## Benchmark

Test setup: An OpenClaw user's memory DB, 251MB after archive cleanup (was 318MB before).

| Version | 1st call | 2nd-5th calls |
|---------|----------|----------------|
| Upstream (default) | ~600ms warm | **~12,000ms each** |
| With PRAGMAs | <1s | **<1s all warm** |

Why: Without `cache_size` + `mmap_size`, SQLite re-reads 64K pages from disk on every cold call. OS page cache is only ~2MB by default.

## Migration

Existing users with large memory DBs should set the three env vars above. No code changes needed — defaults are safe and backward-compatible.
