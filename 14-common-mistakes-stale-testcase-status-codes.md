# Common Mistakes: StaleElement, TestCase, Status Codes, Severity vs Priority

## 1. StaleElementReferenceException

The DOM refreshed after the element was found. The reference points to a dead element.

**When it happens:**
```
FindElement(btn)        ← element found
[AJAX updates the page]
btn.Click()             ← StaleElementReferenceException!
```

**Three causes:**
1. AJAX request updates part of the DOM
2. Navigation — page reloaded
3. Framework re-renders a component (React, Angular)

**Fix — catch inside wait.Until:**
```csharp
wait.Until(d => {
    try {
        d.FindElement(By.Id("submit")).Click();
        return true;
    }
    catch (StaleElementReferenceException) {
        return false; // WebDriverWait retries
    }
});
```

**Key distinction:**
```
StaleElementReferenceException = found, then DOM changed — reference is stale
NoSuchElementException         = element does not exist in the DOM at all
```

---

## 2. [TestCase] — Correct Usage

NOT for different objects/people. FOR different input data to ONE scenario.

```csharp
// CORRECT — same test, different data
[TestCase("admin", "password123", HttpStatusCode.OK)]
[TestCase("admin", "wrongpass",   HttpStatusCode.Forbidden)]
[TestCase("",      "",            HttpStatusCode.Forbidden)]
public void GetToken_ReturnsExpectedStatus(string user, string pass, HttpStatusCode expected)
{
    // one test method, three scenarios
}
```

**When NOT to use it:** if scenarios have fundamentally different steps or logic → separate `[Test]` methods.

---

## 3. POST Status Codes — Common Mistake

```
POST /auth    → 200 OK       ← generates token, does not create a resource
POST /booking → 201 Created  ← creates a new record in the database

Rule:
  POST + new resource in DB → 201
  POST + operation (login, search, calculate) → 200
```

**Mnemonic:** 201 = "1 new record was born"

---

## 4. RestClient in [OneTimeSetUp]

```csharp
[OneTimeSetUp]
public void OneTimeSetup()
{
    client = new RestClient(BaseUrl); // expensive + thread-safe
}
```

**Why [OneTimeSetUp], not [SetUp]:**
- `RestClient` initialization is expensive — creating it before every test wastes time
- `RestClient` is **stateless** — it holds no session, cookies, or auth state between requests
- Because it is stateless, one instance can be safely shared across all tests (no race conditions)

**If it were stateful** (e.g. stored cookies or an auth session), sharing it would be dangerous — tests would bleed state into each other.

---

## 5. Severity vs Priority — The Classic Confusion

```
Severity = HOW BROKEN technically   → set by QA
Priority = HOW URGENTLY to fix      → set by PM/Business
```

| | High Priority | Low Priority |
|---|---|---|
| **High Severity** | Login crash for all users | Crash in rarely-used admin panel |
| **Low Severity** | Typo on homepage | Wrong footer colour |

**Key:** High Severity ≠ High Priority. A crash in an unused module = High Severity, Low Priority.

---

## Interview Q&A

**Q: What is the difference between StaleElement and NoSuchElement?**
A: StaleElement means the element was found but the DOM was updated afterwards — the reference is stale. NoSuchElement means the element does not exist in the DOM at all. Fix for Stale: catch inside wait.Until lambda and return false so polling continues.

**Q: When does POST return 200 instead of 201?**
A: When POST performs an operation rather than creating a resource. POST /auth generates a token — nothing is permanently born in the database, so 200. POST /booking creates a real record — 201.

**Q: Does High Severity automatically mean High Priority?**
A: No. A crash in an admin panel used by 2 people once a month is High Severity (the system is broken) but Low Priority (minimal business impact). Priority is a business decision made by the PM.

**Q: Why is RestClient in [OneTimeSetUp] and not [SetUp]?**
A: Two reasons. First, initialization is expensive — creating a new RestClient before every test wastes time. Second, RestClient is stateless — it holds no session or auth state between requests, so one instance can be safely shared across all tests without race conditions.
