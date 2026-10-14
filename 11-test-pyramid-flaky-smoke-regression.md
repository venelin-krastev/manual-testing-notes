# Test Pyramid, Flaky Tests, Smoke vs Regression

## Test Pyramid

The test pyramid defines the recommended distribution of test types in a test suite.

```
        /\
       /E2E\        ~10%  — slow, brittle, expensive
      /------\
     / Integ. \     ~20%  — test component interactions
    /----------\
   /    Unit    \   ~70%  — fast, isolated, cheap
  /--------------\
```

### Why this shape?

| Level | Speed | Cost | Scope |
|-------|-------|------|-------|
| Unit | Milliseconds | Very low | Single function/method |
| Integration | Seconds | Medium | Multiple components together (e.g. API + DB) |
| E2E | Minutes | High | Full user flow through the UI |

**Rule:** Write as many unit tests as possible. E2E tests are valuable but expensive — they break on UI changes, run slowly, and are hard to debug. Use them only for critical user journeys.

---

## Flaky Tests

A **flaky test** produces inconsistent results — passes sometimes, fails other times — without any code changes.

### Common causes

1. **Timing issues** — `FindElement` runs before the element is rendered. Fix: use `WebDriverWait` instead of `Thread.Sleep`.
2. **Shared state between tests** — one test modifies data that another test depends on. Fix: each test sets up and tears down its own data.
3. **StaleElementReferenceException** — DOM updated after the element reference was captured. Fix: catch inside `wait.Until` lambda and return `false` to keep polling.
4. **Network/environment instability** — external API responds slowly or with intermittent errors.
5. **Order dependency** — tests pass only when run in a specific order. Fix: tests must be independent.

### How to fix flaky tests

```csharp
// Flaky — element may not be loaded yet
driver.FindElement(By.Id("submit")).Click();

// Stable — wait until element is present and clickable
wait.Until(d => d.FindElement(By.Id("submit"))).Click();
```

For `StaleElementReferenceException`:
```csharp
wait.Until(d => {
    try {
        return d.FindElement(By.Id("result")).Text == "Done";
    }
    catch (StaleElementReferenceException) {
        return false; // keep polling
    }
});
```

---

## Smoke Testing vs Regression Testing

| | Smoke | Regression |
|---|---|---|
| **Purpose** | Verify the build is stable enough to test | Verify existing functionality still works after changes |
| **When** | After every new build / deployment | After bug fixes, new features, or refactoring |
| **Scope** | Narrow — critical paths only (login, main flow) | Broad — full feature coverage |
| **Speed** | Fast (minutes) | Slower (hours for full suite) |
| **Analogy** | "Does the car start?" | "Does everything still work after we replaced the engine?" |

**Rule:** Always run smoke first. No point executing the full regression suite if the application doesn't launch.

---

## Interview Q&A

**Q: What is the test pyramid and why does it matter?**
A: The pyramid recommends ~70% unit tests, ~20% integration tests, and ~10% E2E tests. Unit tests are fast, cheap, and isolated — they give fast feedback on individual functions. E2E tests are slow and brittle — they break on UI changes and are expensive to maintain. Inverting the pyramid (more E2E than unit) creates a slow, unreliable suite.

**Q: What is a flaky test? How do you fix it?**
A: A flaky test passes sometimes and fails other times without code changes. Common causes: timing issues (no explicit wait), shared state between tests, or `StaleElementReferenceException`. Fix timing issues with `WebDriverWait`, isolate test data per test, and catch stale element exceptions inside the wait lambda by returning `false` to keep polling.

**Q: When do you run smoke tests vs regression tests?**
A: Smoke tests run after every build to verify the app starts and core flows are functional — if smoke fails, there is no point running anything else. Regression tests run after bug fixes or new features to verify that existing functionality has not broken. Regression scope is much broader than smoke.
