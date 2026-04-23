# Implementation Plan: Token Bucket MiddleWare

## 1. Development Epics
- **Epic 1:** Redis Infrastructure & Lua Scripting
- **Epic 2:** Node.js Middleware Integration
- **Epic 3:** Fallback and Monitoring hooks

## 2. Redis Data Structure
For every unique identifier (e.g., `rate_limit:user_123`), we will store a Hash (`HSET`) in Redis:
- `tokens`: Float (current available tokens)
- `last_refill`: Integer (Unix timestamp in milliseconds)

## 3. The Lua Script (Atomic Execution)
To prevent race conditions (two concurrent requests reading the same token count before either updates it), we MUST execute the check-and-decrement logic atomically inside Redis using a Lua script.

```lua
-- KEYS[1] = rate_limit_key
-- ARGV[1] = bucket_capacity (e.g., 100)
-- ARGV[2] = refill_rate_per_ms
-- ARGV[3] = current_timestamp_ms
-- ARGV[4] = requested_tokens (usually 1)

local tokens_key = 'tokens'
local timestamp_key = 'last_refill'

local bucket_info = redis.call('HMGET', KEYS[1], tokens_key, timestamp_key)
local current_tokens = tonumber(bucket_info[1]) or tonumber(ARGV[1])
local last_refill = tonumber(bucket_info[2]) or tonumber(ARGV[3])

-- Calculate refill
local time_passed = math.max(0, tonumber(ARGV[3]) - last_refill)
local refill_amount = time_passed * tonumber(ARGV[2])

-- Update tokens, cap at bucket capacity
current_tokens = math.min(tonumber(ARGV[1]), current_tokens + refill_amount)

if current_tokens >= tonumber(ARGV[4]) then
    -- Allowed
    current_tokens = current_tokens - tonumber(ARGV[4])
    redis.call('HMSET', KEYS[1], tokens_key, current_tokens, timestamp_key, ARGV[3])
    redis.call('PEXPIRE', KEYS[1], 3600000) -- TTL 1 hour
    return { 1, current_tokens } -- 1 = true
else
    -- Rejected
    redis.call('HMSET', KEYS[1], tokens_key, current_tokens, timestamp_key, ARGV[3])
    return { 0, current_tokens } -- 0 = false
end
```

## 4. HTTP Headers to Implement
The middleware must inject these headers into the response:
- `X-RateLimit-Limit`: The total capacity of the tier.
- `X-RateLimit-Remaining`: Tokens left.
- `X-RateLimit-Reset`: Unix timestamp when the bucket will be completely full.
- `Retry-After`: (Only on 429) Seconds to wait before the next request will succeed.
