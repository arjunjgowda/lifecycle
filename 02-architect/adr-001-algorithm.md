# Architecture Decision Record (ADR) 001: Rate Limiting Algorithm

## 1. Context
We need to select an algorithm to enforce API quotas. Our API sees "bursty" traffic patterns where clients might send 10 concurrent requests, pause for a minute, and send 10 more. 

## 2. Options Considered

### Option A: Fixed Window Counter
- **How it works:** Increments a counter for a given minute (e.g., 09:00 - 09:01).
- **Pros:** Very simple, low memory footprint.
- **Cons:** Boundary issue. A client allowed 100 req/min could send 100 at 09:00:59 and 100 at 09:01:01, effectively doing 200 requests in a 2-second window, overwhelming the backend.

### Option B: Sliding Window Log
- **How it works:** Stores the timestamp of *every* request in a Redis Sorted Set.
- **Pros:** 100% accurate.
- **Cons:** Extremely high memory footprint. A limit of 10,000 req/min requires storing 10,000 timestamps per user. Not scalable.

### Option C: Token Bucket
- **How it works:** A bucket has a maximum capacity (burst limit). Tokens are added to the bucket at a constant rate. A request consumes a token.
- **Pros:** Handles bursts beautifully. Smooths out traffic over time. Very memory efficient (only needs to store 2 values: `current_tokens` and `last_refill_timestamp`).

## 3. Decision
We will use the **Token Bucket** algorithm.

## 4. Implementation Detail: Lazy Refill
To avoid the CPU overhead of a background process constantly refilling millions of buckets, we will use **Lazy Refill**. When a request arrives, we calculate how much time has passed since the `last_refill_timestamp` and mathematically add the appropriate number of tokens before processing the request. This makes the system entirely event-driven.
