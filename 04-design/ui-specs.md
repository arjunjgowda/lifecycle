# UI/UX Specs: Rate Limit States

## 1. Interaction Design
- **Trigger:** User exceeds their allotted quota.
- **Action:** Intercept the outgoing request and prevent it from hitting the server (to save bandwidth).

## 2. Visual States
- **Toast Notification:** A red-themed toast at the top right: "Slow down! You've reached your limit. Try again in 45 seconds."
- **Disabled Buttons:** If the system knows the user is throttled, primary action buttons should become disabled with a tooltip explaining why.

## 3. Empty States
- If a dashboard fails to load due to rate limiting, show a "Quota Exceeded" illustration with a link to "Upgrade to Pro" for higher limits.

## 4. Documentation
- Link to API docs explaining how to use `Retry-After` headers for programmatic users.
