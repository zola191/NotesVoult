## 🔑 Пример использования Keyed Services

> [!info] **Keyed Services** — это функция встроенного DI-контейнера .NET 8+, которая позволяет регистрировать и разрешать несколько реализаций одного интерфейса по строковым или типизированным ключам.

---

### 🧩 Сценарий

Допустим, у нас есть интерфейс `INotificationService`, и две его реализации:
- `EmailNotificationService`
- `SmsNotificationService`

Нам нужно выбирать нужную реализацию в зависимости от типа уведомления.

---

### 1. Определим интерфейс и реализации

```csharp
public interface INotificationService
{
    void Send(string message);
}

public class EmailNotificationService : INotificationService
{
    public void Send(string message) => Console.WriteLine($"Email sent: {message}");
}

public class SmsNotificationService : INotificationService
{
    public void Send(string message) => Console.WriteLine($"SMS sent: {message}");
}
```

---

### 2. Регистрация Keyed Services

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddKeyedScoped<INotificationService>("email", (sp, key) => new EmailNotificationService());
builder.Services.AddKeyedScoped<INotificationService>("sms", (sp, key) => new SmsNotificationService());

var app = builder.Build();
```

---

### 3. Использование через `IServiceProvider` или конструктор

#### В контроллере:

```csharp
app.MapGet("/notify/{type}", (string type, [FromKeyedServices] INotificationService notificationService) =>
{
    notificationService.Send("Hello from " + type);
});
```

> ⚠️ Требуется `.AddEndpointsApiExplorer()` или ASP.NET Core 8+ для атрибута `[FromKeyedServices]`.

Вот переписанный вариант с использованием обычного контроллера вместо Minimal API:

```csharp
[ApiController]
[Route("api/[controller]")]
public class NotificationsController : ControllerBase
{
    private readonly INotificationService _notificationService;

    public NotificationsController(
        [FromKeyedServices("email")] INotificationService notificationService)
    {
        _notificationService = notificationService;
    }

    [HttpGet("{type}")]
    public IActionResult Notify(string type)
    {
        _notificationService.Send($"Hello from {type}");
        return Ok($"Notification sent via {type}");
    }
}
```

Или более гибкий вариант, где сервис выбирается динамически:

```csharp
[ApiController]
[Route("api/[controller]")]
public class NotificationsController : ControllerBase
{
    private readonly IServiceProvider _serviceProvider;

    public NotificationsController(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    [HttpGet("{type}")]
    public IActionResult Notify(string type)
    {
        var service = _serviceProvider.GetRequiredKeyedService<INotificationService>(type);
        service.Send($"Hello from {type}");
        return Ok($"Notification sent via {type}");
    }
}
```

Для работы этих примеров нужно:

1. Убедиться, что у вас .NET 8+
2. Добавить в Program.cs регистрацию контроллера:
```csharp
builder.Services.AddControllers();
// ...
app.MapControllers();
```

3. Keyed-сервисы должны быть зарегистрированы как показано в оригинальном примере.
---

#### Либо через `IServiceProvider` напрямую:

```csharp
var emailService = app.Services.GetRequiredKeyedService<INotificationService>("email");
emailService.Send("Test email");
```

---

### ✅ Преимущества Keyed Services

- Избавляет от необходимости использовать `IEnumerable<T>` и фильтрацию по типу.
- Удобен при множестве реализаций одного интерфейса.
- Чистый и понятный API.
- Поддерживает внедрение через конструктор с использованием `[FromKeyedServices]`.

---

### ❌ Минусы

- Доступен только начиная с **.NET 8**.
- Нет поддержки ручной регистрации в старых версиях.
- Меньше гибкости по сравнению с `Microsoft.Extensions.DependencyInjection.Abstractions` и `Func<string, T>` фабриками (вручную).

---

### 📝 Альтернатива до .NET 8

Если вы не на .NET 8, можно использовать `Func<string, T>`:

```csharp
public interface INotificationServiceFactory
{
    INotificationService Create(string key);
}

public class NotificationServiceFactory : INotificationServiceFactory
{
    private readonly IServiceProvider _provider;

    public NotificationServiceFactory(IServiceProvider provider) => _provider = provider;

    public INotificationService Create(string key)
    {
        return key switch
        {
            "email" => _provider.GetRequiredService<EmailNotificationService>(),
            "sms" => _provider.GetRequiredService<SmsNotificationService>(),
            _ => throw new ArgumentOutOfRangeException()
        };
    }
}
```

---

### 🔗 Полезные ссылки

- [Microsoft Docs: Keyed Services](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#keyed-services)
- [ASP.NET Core 8 Dependency Injection Updates](https://devblogs.microsoft.com/dotnet/announcing-asp-net-core-in-dotnet-8/#keyed-dependency-injection)
