# ADR 001: Selection of Rate Limiting Algorithm

## Status
Accepted

## Context
We need an algorithm that handles distributed traffic and allows for small bursts of activity from legitimate users without blocking them immediately.

## Decision
We will use the **Token Bucket** algorithm.

## Alternatives Considered
1. **Fixed Window Counter:** Rejected due to "boundary problems" where double the limit can be served at the edge of two windows.
2. **Leaky Bucket:** Rejected because it smooths out traffic too aggressively, which might hurt user experience during legitimate short spikes.

## Consequences
- **Pros:** Memory efficient, handles bursts well.
- **Cons:** Slightly more complex to implement (requires timestamp tracking).
