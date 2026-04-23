# Product Requirements Document (PRD): Global API Rate Limiting

## 1. Overview and Problem Statement
Our public APIs are currently vulnerable to abuse, "noisy neighbor" problems, and potential Distributed Denial of Service (DDoS) attacks. A single client can saturate our database connections, degrading performance for all other clients. We need to implement a tier-based rate limiting system to ensure fair usage and protect system stability.

## 2. Business Objectives
- **System Stability:** Prevent localized spikes in traffic from causing platform-wide outages.
- **Fair Usage:** Ensure resources are distributed according to the customer's subscription tier.
- **Monetization Support:** Create a clear distinction between Free and Paid tiers based on API throughput limits.

## 3. Scope
**In Scope:**
- Rate limiting for all public-facing REST API endpoints (`/api/v1/*`).
- Identification via `Authorization` header (Bearer token) or Client IP (for unauthenticated routes).
- Graceful degradation returning HTTP `429 Too Many Requests`.

**Out of Scope:**
- Throttling internal service-to-service communication.
- Web Application Firewall (WAF) layer protections (e.g., SQLi blocking) - handled by Cloudflare.

## 4. User Tiers & Entitlements
| Tier | Limit (Requests) | Window | Burst Capacity (Bucket Size) |
| :--- | :--- | :--- | :--- |
| Unauthenticated | 60 | 1 Minute | 10 requests |
| Free Tier | 1,000 | 1 Hour | 50 requests |
| Pro Tier | 10,000 | 1 Minute | 500 requests |
| Enterprise | Custom | Custom | Custom (defined in contract) |

## 5. User Experience (UX)
- Clients exceeding the limit MUST receive an immediate `429` response.
- The response MUST include a clear error message.
- The response MUST include standard headers indicating when they can retry.

## 6. Success Metrics
- **Reliability:** 0 dropped database connections due to client flooding.
- **Performance:** Rate limiter evaluation adds < 5ms to the request lifecycle (P99).
- **Accuracy:** Limit enforcement has < 1% error margin under high concurrency.
