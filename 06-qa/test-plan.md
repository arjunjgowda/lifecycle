# QA Test Plan: Rate Limiter Middleware

## 1. Functional Testing (Postman/Newman)

### Scenario A: Free Tier Limits
- **Given** a user on the Free tier (100 req/min, capacity 10).
- **When** the user sends 10 requests instantly.
- **Then** all 10 receive `200 OK`. The 10th request shows `X-RateLimit-Remaining: 0`.
- **When** the user sends an 11th request immediately.
- **Then** it receives `429 Too Many Requests`.
- **When** the user waits exactly 600ms (100req/60sec = 1 req per 0.6s) and sends a request.
- **Then** it receives `200 OK` (verifies lazy refill is working continuously).

## 2. Load and Concurrency Testing (K6)
We must ensure the Lua script successfully prevents race conditions.

### Scenario B: Concurrent Execution Bypass Attempt
- **Script:** Create a `k6` script that uses `http.batch` to send 50 requests simultaneously from the *exact same API key* against a limit of 10.
- **Expected Result:** Exactly 10 requests must return `200 OK`. Exactly 40 requests must return `429`. If more than 10 return 200, we have a race condition in Redis.

## 3. Resilience Testing (Chaos Engineering)

### Scenario C: Redis Failure
- **Action:** Temporarily block port 6379 (Redis) on the API Gateway.
- **Expected Result:** The middleware's "Fail Open" logic triggers. Requests should take ~5ms longer (timeout), but should return `200 OK` from the backend, not `500 Internal Server Error`. A warning should be logged.
