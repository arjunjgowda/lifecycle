# UI/UX Specifications: Rate Limiting Error States

## 1. API Response Format
When a request is blocked, the server must return HTTP Status `429 Too Many Requests`. The JSON body must follow the standard error schema:

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "You have exceeded your API quota of 1000 requests per hour.",
    "details": {
      "retry_after_seconds": 45,
      "upgrade_url": "https://dashboard.example.com/billing"
    }
  }
}
```

## 2. Frontend Application Handling
The frontend application (React/Vue) must implement a global Axios/Fetch interceptor to catch `429` responses.

**Behavior:**
1. **Toast Notification:** Display a warning toast: `"Rate limit reached. Pausing requests for X seconds."` (Parse X from the `Retry-After` header).
2. **Exponential Backoff:** If the frontend is making automated polling requests, it must halt polling and wait exactly the duration specified in the `Retry-After` header before resuming.
3. **Button Disabling:** If the user triggered an action (e.g., clicking "Save" repeatedly), disable the button and show a countdown timer inside the button.

## 3. Developer Portal Documentation
Update the public documentation portal to explain the Token Bucket algorithm to end-users so they understand that they don't have to wait for a full hour to reset; tokens "trickle" back in continuously. Include examples of reading the `X-RateLimit-*` headers in Python and Node.js.
