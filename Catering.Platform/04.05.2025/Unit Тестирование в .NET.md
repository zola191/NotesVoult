# 🧪 Unit Тестирование в .NET

> [!summary] **Unit тестирование** — это процесс проверки отдельных модулей или методов изолированно от остальной системы.  
Цель: убедиться, что каждый модуль работает корректно в изоляции.

---

## 🔧 Библиотеки для тестирования

### Основные фреймворки:
| Фреймворк | Особенности |
|-----------|-------------|
| **xUnit** | Современный, активно развивается, рекомендован Microsoft |
| **NUnit** | Мощный, гибкий, поддерживает параметризованные тесты |
| **MSTest** | Встроенный в Visual Studio, простой для новичков |

### Вспомогательные библиотеки:
| Библиотека | Назначение |
|------------|------------|
| **Moq** | Mocking framework, поддерживает лямбда-выражения |
| **NSubstitute** | Простой и читаемый синтаксис моков |
| **FakeItEasy** | Удобен при BDD-подходе |
| **JustMock** | Расширенный функционал, платный |
| **Microsoft Fakes** | Инструмент шунтирования, часть Visual Studio Enterprise |

---

## 📐 Структуры написания тестов

| Паттерн | Описание |
|--------|----------|
| **AAA (Arrange-Act-Assert)** | Подготовка → действие → проверка |
| **GWT (Given-When-Then)** | Используется в BDD, описывает поведение на естественном языке |
| **SEVT (Setup-Exercise-Verify-TearDown)** | Классическая структура |
| **CSE (Context/Specification)** | Для BDD и функциональных языков |
| **Method Pattern** | Чёткие названия по методу и сценарию |

---

### Пример GWT (BDD):

> [!example] Gherkin-сценарий
```gherkin
Feature: Calculator
  In order to avoid mistakes
  As a user
  I want to use a calculator to add numbers

  Scenario: Add two numbers
    Given I have entered 50 into the calculator
    And I have entered 70 into the calculator
    When I press add
    Then the result should be 120
```

> [!example] Реализация на C# (SpecFlow)
```csharp
[Binding]
public class CalculatorSteps
{
    private Calculator _calculator = new();
    private int _result;

    [Given(@"I have entered (.*) into the calculator")]
    public void GivenIHaveEnteredNumber(int number)
    {
        _calculator.Enter(number);
    }

    [When(@"I press add")]
    public void WhenIPressAdd()
    {
        _result = _calculator.Add();
    }

    [Then(@"the result should be (.*)")]
    public void ThenTheResultShouldBe(int expected)
    {
        Assert.Equal(expected, _result);
    }
}
```

> [!warning] Недостатки BDD:
- Требует поддержки шагов (`step definitions`)
- Активное использование рефлексии может снижать производительность

---

## 🤖 Mock vs Stub

| Тип | Цель | Как использовать |
|-----|------|------------------|
| **Stub** | Возвращает фиксированные данные | `Setup(...).Returns(...)` в Moq / `Returns(...)` в NSubstitute |
| **Mock** | Проверяет взаимодействие (например, вызов метода) | `Verify(...)` в Moq / `Received()` в NSubstitute |

---

## ✅ Пример юнит-теста с Moq и NSubstitute

Допустим, есть класс `OrderService`, зависящий от `IInventoryService`.

```csharp
public interface IInventoryService
{
    bool IsInStock(int productId, int quantity);
    void ReserveItems(int productId, int quantity);
}

public class OrderService
{
    private readonly IInventoryService _inventory;

    public OrderService(IInventoryService inventory) => _inventory = inventory;

    public bool PlaceOrder(int productId, int quantity)
    {
        if (!_inventory.IsInStock(productId, quantity))
            return false;

        _inventory.ReserveItems(productId, quantity);
        return true;
    }
}
```

---

### > [!example] Тест с Moq

```csharp
[Fact]
public void PlaceOrder_WhenInStock_ReservesItems()
{
    // Arrange
    var mockInventory = new Mock<IInventoryService>();
    mockInventory.Setup(i => i.IsInStock(1, 2)).Returns(true);

    var service = new OrderService(mockInventory.Object);

    // Act
    var result = service.PlaceOrder(1, 2);

    // Assert
    Assert.True(result);
    mockInventory.Verify(i => i.ReserveItems(1, 2), Times.Once());
}
```

---

### > [!example] Тест с NSubstitute

```csharp
[Fact]
public void PlaceOrder_WhenInStock_ReservesItems()
{
    // Arrange
    var inventory = Substitute.For<IInventoryService>();
    inventory.IsInStock(1, 2).Returns(true);

    var service = new OrderService(inventory);

    // Act
    var result = service.PlaceOrder(1, 2);

    // Assert
    Assert.True(result);
    inventory.Received().ReserveItems(1, 2);
}
```

---

## ⚖️ Сравнение Moq и NSubstitute

| Возможность | Moq | NSubstitute |
|-------------|-----|--------------|
| Создание мока | `new Mock<IService>()` | `Substitute.For<IService>()` |
| Настройка возврата значения | `.Setup(x => x.Method()).Returns(value)` | `.Method().Returns(value)` |
| Проверка вызова | `.Verify(x => x.Method(), Times.Once())` | `.Received().Method()` |
| Читаемость | Сложнее для новичков | Более интуитивно понятен |
| Поддержка BDD | Хорошая | Отличная, особенно с NSpec |

---

## 🚨 Тестирование исключений

> [!example]
```csharp
[Fact]
public void Divide_WhenDividedByZero_ThrowsException()
{
    // Arrange
    var calculator = new Calculator();

    // Act & Assert
    Assert.Throws<DivideByZeroException>(() => calculator.Divide(5, 0));
}
```

---

## 🏗️ Builder Pattern для объектов

Полезен для создания сложных объектов в тестах.

```csharp
public class CustomerBuilder
{
    private string _name = "John Doe";
    private int _age = 30;

    public Customer Build() => new(_name, _age);

    public CustomerBuilder WithName(string name)
    {
        _name = name;
        return this;
    }

    public CustomerBuilder WithAge(int age)
    {
        _age = age;
        return this;
    }
}
```

Пример использования:
```csharp
var customer = new CustomerBuilder().WithName("Alice").WithAge(25).Build();
```

---

## ❌ Частые ошибки при тестировании

> [!danger] Не делай так:
- **Мокаешь всё подряд**, даже константы.
- **Проверяешь внутренние реализации**, а не поведение.
- **Используешь `Thread.Sleep()`**, чтобы дать время чему-то выполниться.
- **Зависишь от времени (`DateTime.Now`)** — используй абстракцию `IClock`.
- **Тестируешь DTO, Enum, Razor/XAML** — там нет логики.
- **Тестируешь приватные методы** — выноси в публичные сервисы.

---

## 🧩 Параметризованные тесты (Tabular tests)

### xUnit
> [!example]
```csharp
[Theory]
[InlineData(2, 3, 5)]
[InlineData(-1, 1, 0)]
public void Add_TwoNumbers_ReturnsSum(int a, int b, int expected)
{
    Assert.Equal(expected, a + b);
}
```

### NUnit
> [!example]
```csharp
[Test]
public void Add_TwoNumbers_ReturnsSum(
    [Values(2, -1)] int a, 
    [Values(3, 1)] int b,
    [Values(5, 0)] int expected)
{
    Assert.AreEqual(expected, a + b);
}
```

---

## 🛠 Советы и вопросы для развития

- **DTO: `record` vs `class`** — предпочтение отдавать `record`, особенно для неизменяемых данных.
- **Исключения в контроллерах**: использовать middleware вместо try-catch.
- **Keyed Services** — можно регистрировать через DI контейнер с одним интерфейсом.
- **HATEOAS** — возвращать ссылки в REST API, если требуется.
- **Логирование**:
  - `Debug` — детальные сообщения
  - `Info` — общее состояние
  - `Error` — ошибки приложения

---

## 📦 Чек-лист перед написанием теста

✅ Повысит ли тест надёжность приложения?  
✅ Обнаружит ли он регрессию при изменении логики?  
✅ Упростит ли отладку?  

---

## 📚 Полезные ссылки

- [xUnit.net](https://xunit.net/)
- [Moq Quickstart](https://github.com/moq/moq4/wiki/Quickstart)
- [NSubstitute Docs](https://nsubstitute.github.io/)
- [FluentAssertions Docs](https://fluentassertions.com/)
- [SpecFlow](https://specflow.org/)
- [Coverlet](https://github.com/coverlet-coverage/coverlet)
- [TDD by Martin Fowler](https://martinfowler.com/bliki/TestDrivenDevelopment.html)
