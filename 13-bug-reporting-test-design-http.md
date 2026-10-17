# Bug Reporting, Test Case Design & HTTP Basics

## Bug Reporting

A good bug report lets the developer reproduce the issue without asking any questions.

### Required Fields

| Field | What to write |
|-------|--------------|
| **Title** | Specific: "Login button unresponsive on Safari 17 with autofill" |
| **Environment** | Browser, OS, version, device |
| **Preconditions** | System state required before starting |
| **Steps to Reproduce** | Numbered, minimal — fewest steps to trigger the bug |
| **Expected Result** | What should happen |
| **Actual Result** | What actually happens |
| **Severity** | Critical / High / Medium / Low |
| **Attachments** | Screenshot or screen recording |

### Example Bug Report

```
Title: Login button unresponsive after entering valid credentials on Safari 17

Environment: Safari 17.2 / macOS Sonoma / Desktop

Preconditions: User is registered with email test@mail.com

Steps to Reproduce:
1. Navigate to https://example.com/login
2. Enter email: test@mail.com
3. Enter password: Test1234!
4. Click "Login" button

Expected: User is redirected to /dashboard
Actual: Button click has no effect; no error message, no redirect, no network request

Severity: High
Attachments: [screen_recording.mp4]
```

---

## Severity vs Priority

```
Severity  = HOW BROKEN (technical impact) → set by QA
Priority  = HOW URGENT to fix             → set by business/PM
```

| Example | Severity | Priority |
|---------|----------|----------|
| Typo on homepage | Low | High — brand visibility |
| Crash in unused admin panel | High | Low |
| Payment fails for all users | Critical | Critical |
| Wrong button colour | Low | Low |

---

## Defect Lifecycle

```
New → Assigned → Open → Fixed → Retest → Closed
                                    ↓
                                Reopened  (fix didn't work)

Special statuses:
  Rejected  — not a bug; expected behaviour
  Deferred  — valid bug, postponed to future sprint
```

---

## Test Case Design

### Equivalence Partitioning (EP)

Group inputs where all values in a group behave the same. Test **one value per group**.

```
Age field:
  < 0        → invalid  → test: -1
  0–17       → invalid  → test: 10
  18–65      → valid    → test: 30
  66–120     → invalid  → test: 70
  > 120      → invalid  → test: 150

Result: 5 tests instead of infinite
```

### Boundary Value Analysis (BVA)

Test the **edges** of partitions — bugs cluster at boundaries.

```
Valid range 18–65:
  test: 17, 18, 65, 66   ← boundaries
  NOT: 30 (middle)       ← already covered by EP
```

**Use both together:** EP defines the groups, BVA tests the edges of each group.

### Decision Table (bonus)

For combinations of conditions:

```
| Username ✓ | Password ✓ | Result           |
|------------|------------|------------------|
| valid      | valid      | Login OK         |
| valid      | invalid    | "Wrong password" |
| invalid    | valid      | "User not found" |
| invalid    | invalid    | "User not found" |
```

---

## HTTP Methods

```
GET     — retrieve, no body
POST    — create, body required
PUT     — full replace (send ALL fields)
PATCH   — partial update (send only changed fields)
DELETE  — remove, usually no body
```

### PUT vs PATCH difference

```
Booking: { firstname: "Alice", lastname: "Smith", price: 100 }

PUT   { firstname: "Bob" }  → { firstname: "Bob", lastname: null, price: null }
PATCH { firstname: "Bob" }  → { firstname: "Bob", lastname: "Smith", price: 100 }
```

PUT replaces the entire resource. PATCH updates only what you send.

---

## HTTP Status Codes

```
2xx — Success
  200 OK           — standard success
  201 Created      — resource created (response to POST)

4xx — Client error
  400 Bad Request       — invalid input from client
  401 Unauthorized      — not authenticated
  403 Forbidden         — authenticated but no permission
  404 Not Found         — resource does not exist
  405 Method Not Allowed — HTTP method not supported on this endpoint

5xx — Server error
  500 Internal Server Error — server-side failure
```

Always assert the **exact** status code in tests, not just "2xx".

---

## Interview Q&A

**Q: What makes a good bug report?**
A: The developer must be able to reproduce it without asking any questions. Minimal numbered steps, exact environment, clear expected vs actual, and a screen recording. A vague title like "login doesn't work" is useless — "Login button unresponsive on Safari 17 with autofill active" tells the developer exactly where to look.

**Q: What is the difference between severity and priority?**
A: Severity is the technical impact — how badly the bug breaks the system. Priority is business urgency — how soon it needs to be fixed. QA sets severity; business/PM sets priority. A typo on the homepage is low severity but high priority because of brand visibility.

**Q: What is the difference between equivalence partitioning and boundary value analysis?**
A: EP divides inputs into groups with the same expected behaviour — you test one value per group. BVA tests the edges of those groups, because bugs cluster at boundaries. You use both together: EP to define the groups, BVA to test the edges of each.

**Q: What is the difference between PUT and PATCH?**
A: PUT replaces the entire resource — if you omit a field, it becomes null. PATCH updates only the fields you send — the rest stay unchanged. In restful-booker, partial updates use PATCH, not PUT.

**Q: What are the most important HTTP status codes for API testing?**
A: 200 (success), 201 (created), 400 (bad input), 401 (not authenticated), 403 (no permission), 404 (not found), 405 (method not allowed), 500 (server error). Always assert the exact code — 404 and 405 mean very different things even though both are 4xx.
