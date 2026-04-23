# Implementation Plan: Token Bucket (Lazy Refill)

## 1. Technical Strategy
We will use the **Lazy Refill** strategy to avoid the overhead of background cron jobs. Tokens will be replenished mathematically upon every request.

## 2. Task Breakdown (Sprint 42)
- **Ticket-101:** Create Redis Lua script for atomic `Get-Calculate-Update` operations. (Assigned: Dev-A)
- **Ticket-102:** Implement Rate Limiter Middleware for Express.js. (Assigned: Dev-B)
- **Ticket-103:** Add `429` error handling to Global Exception Handler. (Assigned: Dev-C)

## 3. Data Integrity
- Ensure `tokens_to_add` is calculated using high-precision timestamps (micro-seconds) to avoid drift.
- Use `WATCH` or Lua scripts in Redis to prevent race conditions during concurrent requests.

## 4. Definition of Done (DoD)
- Unit tests pass with >90% coverage.
- Middleware adds < 2ms latency on local benchmarks.
- No tokens are "leaked" (lost) during high concurrency.
