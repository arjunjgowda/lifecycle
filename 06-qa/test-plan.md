# QA Test Plan: Rate Limiting

## 1. Positive Testing
- Verify that a user under the limit (e.g., 50/100) receives a `200 OK`.
- Verify `X-RateLimit-Remaining` header decrements correctly.

## 2. Negative Testing (Throttling)
- Send 101 requests in 1 minute for a 100/min limit.
- Verify 101st request returns `429 Too Many Requests`.
- Verify `Retry-After` header is present and accurate.

## 3. Burst Testing
- Send 20 requests simultaneously (instantaneous).
- Verify the "Bucket Capacity" handles the burst if configured to do so.

## 4. Performance/Distributed Testing
- Run load tests from 5 different geographical regions.
- Ensure the Redis sync is fast enough that the limit is global, not local to the node.
