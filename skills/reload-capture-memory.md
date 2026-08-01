---
name: Capture and recall shared Memory
description: Write a decision or fact into Reload's shared context graph with provenance, link and supersede it correctly, and recall related context later.
api: openapi/reload-openapi-original.json
operations: [remember-memory, recall, search-memories, link-nodes, supersede-memory, invalidate-memory, flag-contradiction, resolve-identity]
---

# Capture and recall shared Memory

Reload Memory is a shared context graph agents use to remember decisions and
share facts across tools. Use it instead of dumping transcripts into a prompt.

## Auth
`Authorization: Bearer <rl_sk_ key>`. Memory primitives live under `/v1/sdk/*`
and take **snake_case** fields (`scope_id`, `derived_from`, `expected_version`).

## Steps
1. **Author a memory** with `remember-memory`. Provide the `kind`
   (decision / fact / preference), the content, a `scope_id`, and **at least one
   `derived_from` provenance pointer** — a memory without provenance is rejected.
   If you only have a person's handle/email, resolve their id first with
   `resolve-identity` for the `stated_by` field.
2. **Link related nodes** with `link-nodes` — pass `to_type` explicitly when the
   target is not a Memory, or the edge silently lands as Memory→Memory.
3. **Recall later** with `recall`: pass exactly one of `query` (semantic ANN) or
   `seed_id` (neighborhood walk). It returns a ranked subgraph with edges and
   provenance. Use `search-memories` for a structured filter (kind / status /
   tags / date / scope).
4. **Evolve a memory** with `supersede-memory` (new version, guarded by
   `expected_version`) or retire it with `invalidate-memory`. On a
   `409 VERSION_CONFLICT`, re-read for the current version and retry.
5. **Record a conflict** between two memories with `flag-contradiction` (writes
   a CONTRADICTS edge with a note).

## Rules
- Recall is ACL-bounded by scope membership — memories from channels you cannot
  see are never returned.
- Optimistic locking, not idempotency keys: every write that mutates an existing
  node passes `expected_version`. See `conventions/reload-conventions.yml`.
