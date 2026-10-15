# StaleElement, FindElements, and Driver Lifecycle

## StaleElementReferenceException

A `StaleElementReferenceException` is thrown when a reference to a DOM element becomes invalid because the DOM was updated after the element was found.

### Cause

```
1. driver.FindElement(By.Id("result"))  ← element reference captured
2. Page reloads or DOM updates          ← reference becomes stale
3. .Text or .Click() on old reference  ← StaleElementReferenceException thrown
```

### Fix — catch inside wait.Until lambda

```csharp
wait.Until(d => {
    try {
        return d.FindElement(By.Id("result")).Text == "Done";
    }
    catch (StaleElementReferenceException) {
        return false; // keep polling until DOM stabilises
    }
});
```

**Why this works:** `wait.Until` keeps polling until the lambda returns `true`. If a stale reference is caught, returning `false` triggers the next poll — re-finding the element with a fresh reference each time.

---

## FindElement vs FindElements

| | `FindElement` | `FindElements` |
|---|---|---|
| **Returns** | Single `IWebElement` | `IReadOnlyCollection<IWebElement>` |
| **If nothing found** | Throws `NoSuchElementException` | Returns empty collection (no exception) |
| **Use case** | You expect exactly one element | Checking existence or counting elements |

```csharp
// Safe existence check — no exception if not found
var errors = driver.FindElements(By.CssSelector(".error-message"));
if (errors.Count > 0)
    Console.WriteLine(errors[0].Text);
```

---

## Driver Lifecycle — SetUp vs TearDown

### Why fresh driver per test?

```csharp
[SetUp]
public void Setup()
{
    driver = DriverFactory.Create(); // fresh browser per test
}

[TearDown]
public void Teardown()
{
    driver?.Quit();    // closes browser + ends ChromeDriver process
    driver?.Dispose(); // frees managed resources
}
```

**Both are needed:**
- `Quit()` — closes the browser window and kills the ChromeDriver process
- `Dispose()` — releases the managed .NET wrapper object

Calling only `Quit()` leaks the .NET object. Calling only `Dispose()` leaves a zombie ChromeDriver process running.

### Why RestClient in OneTimeSetUp?

```csharp
[OneTimeSetUp]
public void OneTimeSetup()
{
    client = new RestClient(BaseUrl); // created once per test class
}

[OneTimeTearDown]
public void OneTimeTeardown()
{
    client?.Dispose();
}
```

**RestClient is:**
- **Expensive to initialise** — creates connection pools and HTTP handlers
- **Thread-safe and stateless** — no test bleeds state into another via the client
- **Shared safely** — one instance per test class is correct

`[SetUp]` would create a new `RestClient` before every test — wasteful and slower with no benefit.

---

## Interview Q&A

**Q: What causes StaleElementReferenceException and how do you fix it?**
A: The DOM updates after the element reference was captured — for example, after a page reload or dynamic content change. The reference points to a node that no longer exists in the current DOM. The fix is to re-find the element on every poll by placing `FindElement` inside a `wait.Until` lambda and catching `StaleElementReferenceException` to return `false`, which keeps the polling going until a fresh reference succeeds.

**Q: What is the difference between FindElement and FindElements?**
A: `FindElement` returns a single element and throws `NoSuchElementException` if nothing matches. `FindElements` returns an `IReadOnlyCollection` — if nothing matches, it returns an empty collection without throwing. Use `FindElements` when you need to check existence or count elements safely.

**Q: Why do you call both Quit() and Dispose() in TearDown?**
A: `Quit()` closes the browser and terminates the ChromeDriver process. `Dispose()` releases the managed .NET wrapper. You need both — `Quit()` alone leaks the managed object, `Dispose()` alone leaves a background ChromeDriver process running.

**Q: Why is RestClient in OneTimeSetUp instead of SetUp?**
A: RestClient is expensive to initialise — it creates connection pools and HTTP handlers. It is also thread-safe and stateless, so there is no risk of test contamination from sharing one instance. Creating it fresh before every test wastes resources with no correctness benefit.
