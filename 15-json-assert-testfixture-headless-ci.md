# JSON Assert, [TestFixture], Headless CI Debug

## 1. Assertване на JSON поле с RestSharp + NUnit

```csharp
var response = client.Execute(request);

var body = JObject.Parse(response.Content!);

Assert.That(body["firstname"]?.ToString(), Is.EqualTo("Jim"),
    "firstname field should be Jim");
```

**Защо `?.ToString()`:**
- `body["firstname"]` връща `JToken`, не `string`
- Без `.ToString()` → type mismatch при Assert дори ако стойността изглежда еднаква
- `?.` пази от `NullReferenceException` ако полето липсва в response-а

---

## 2. [TestFixture] — какво прави

```csharp
[TestFixture]
public class LoginTests
{
    [Test]
    public void Should_Login() { }
}
```

**Какво прави:** маркира класа като test class — NUnit го включва в test runner-а.

**Ако го махнеш в NUnit 3+:** тестовете пак минават (NUnit 3 открива публични класове автоматично). Но:
- По-стари CI конфигурации може да не го открият
- `abstract` или `internal` клас без `[TestFixture]` → NUnit го игнорира изцяло

**Правило:** винаги слагай `[TestFixture]` — конвенция > "работи и без него".

---

## 3. Headless CI фейл — диагноза

```
Локално минава + CI фейлва с NoSuchElement = три подозрения
```

| Причина | Fix |
|---------|-----|
| Timing — елементът се зарежда бавно | `WebDriverWait` |
| Viewport — headless default е малък (800x600) | `--window-size=1920,1080` |
| Забравен файл — локален ресурс не е в git | `git status` преди push |
| Различна среда — staging ≠ local URL | Провери env vars |

**Screenshot on failure за CI debug:**
```csharp
[TearDown]
public void Teardown()
{
    if (TestContext.CurrentContext.Result.Outcome.Status == TestStatus.Failed)
    {
        var screenshot = ((ITakesScreenshot)driver).GetScreenshot();
        screenshot.SaveAsFile($"screenshot_{TestContext.CurrentContext.Test.Name}.png");
    }
    driver?.Quit();
    driver?.Dispose();
}
```

После в GitHub Actions: upload като artifact → виждаш точно какво е на екрана при фейл.

---

## Interview Q&A

**Q: Тестът минава локално, фейлва в CI headless. Как дебъгваш?**
A: Гледам CI лога за точния ред. Проверявам трите най-чести причини: timing (WebDriverWait), viewport размер (--window-size=1920,1080), забравен файл. Репродуцирам локално с `CI=true` и добавям screenshot on failure за визуален контекст.

**Q: Защо `body["firstname"]?.ToString()` вместо директно `body["firstname"]`?**
A: `body["firstname"]` връща `JToken`, не `string`. Assert ще сравни `JToken` с `string` — type mismatch дори при еднаква стойност. `.ToString()` конвертира. `?.` пази от `NullReferenceException` ако полето липсва.

**Q: Какво прави [TestFixture] и какво се случва без него?**
A: Маркира класа като test class за NUnit. В NUnit 3+ тестовете минават и без него (auto-discovery), но по-стари CI инструменти може да не открият класа. `abstract` и `internal` класове без него се игнорират изцяло.
