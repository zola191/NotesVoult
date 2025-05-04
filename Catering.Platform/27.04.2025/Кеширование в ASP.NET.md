## Что такое кеширование?

**Кеширование** — это процесс сохранения данных в быстром хранилище (кеше), чтобы снизить нагрузку на БД и ускорить обработку запросов.  
Кеширование позволяет **повторно использовать уже полученные или вычисленные данные**, минуя дорогостоящие операции (например, обращение к базе данных).

---

## 4 типа кеширования в ASP.NET

1. **In-memory Cache**
2. **Распределённый кэш (Redis)**
3. **Response Caching**
4. **Tag Helper Caching (в Razor)**

---

## 1. In-Memory Cache

### Описание
Хранится в памяти текущего процесса. Самый простой и быстрый способ кеширования.

### Подключение
```csharp
builder.Services.AddMemoryCache();
```

### Пример использования
```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;

    public ProductService(IMemoryCache cache)
    {
        _cache = cache;
    }

    public Product GetProduct(int id)
    {
        return _cache.GetOrCreate($"product_{id}", entry =>
        {
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
            return LoadProductFromDatabase(id);
        });
    }
}
```

### Сценарии использования
- Небольшие приложения или прототипирование.
- Отсутствие горизонтального масштабирования.
- Данные, которые часто читаются и редко меняются.

### Минусы
- При перезапуске приложения кэш теряется.
- Не поддерживает распределённое кеширование.
- Может привести к увеличению потребления памяти.
- Риск утечки памяти при неправильном управлении.

---

## 2. Распределённый кэш (Redis)

### Описание
Кэш, доступный из нескольких экземпляров приложения. Часто используется Redis.

### Подключение
```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
});
```

> ⚠️ Для продакшена рекомендуется указать параметры повторных попыток и таймаутов:
```csharp
options.ConfigurationOptions = new ConfigurationOptions
{
    EndPoints = { "redis-host:6379" },
    ConnectRetry = 3,
    AbortOnConnectFail = false,
    DefaultDatabase = 0
};
```

### Пример использования
```csharp
public class DistributedProductService
{
    private readonly IDistributedCache _cache;

    public DistributedProductService(IDistributedCache cache)
    {
        _cache = cache;
    }

    public async Task<string> GetProductNameAsync(int id)
    {
        var name = await _cache.GetStringAsync($"product_{id}");
        if (name != null)
            return name;

        var productName = $"Product {id}";
        await _cache.SetStringAsync($"product_{id}", productName,
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            });

        return productName;
    }
}
```

### Сценарии использования
- Облачная архитектура.
- Микросервисная архитектура.
- Высокая отказоустойчивость.
- Горизонтальное масштабирование.

### Минусы
- Зависимость от сети и внешней инфраструктуры.
- Усложнённая настройка и мониторинг.
- Дополнительные затраты на обслуживание Redis.
- Задержки при сетевых вызовах.

---

## 3. Response Caching

### Описание
Кеширование HTTP-ответов на уровне контроллеров. Использует заголовки `Cache-Control`, `ETag`, `Vary`.

### Подключение
```csharp
builder.Services.AddResponseCaching();
app.UseResponseCaching();
```

### Пример использования
```csharp
[HttpGet]
[ResponseCache(Duration = 60)]
public IActionResult GetWeather()
{
    return Ok(new { Temperature = "22C", Condition = "Sunny" });
}
```

### Сценарии использования
- Статичные или редко изменяемые данные (расписания, справочники).
- API, возвращающие одинаковые ответы при одних и тех же входных данных.

### Минусы
- Не подходит для персонализированных данных.
- Требуется аккуратная настройка заголовков.
- Нет механизма ручной инвалидации кэша.

### Советы
- Используйте `VaryByHeader` для кэширования разных версий ответа:
```csharp
[ResponseCache(Duration = 60, VaryByHeader = "User-Agent")]
```
- Используйте `ETag` для оптимизации трафика:
```csharp
[HttpGet]
public IActionResult GetData()
{
    var data = GetFromDb();
    var eTag = CalculateETag(data); // Например, хэш объекта

    if (Request.Headers.IfNoneMatch == eTag)
        return StatusCode(StatusCodes.Status304NotModified);

    Response.Headers.ETag = eTag;
    return Ok(data);
}
```

---

## 4. Tag Helper Caching (в Razor)

### Описание
Кеширование HTML-фрагментов на стороне сервера в Razor.

### Пример использования
```cshtml
<cache expires-after="TimeSpan.FromMinutes(5)">
    <h2>Текущая дата: @DateTime.Now</h2>
</cache>
```

### Сценарии использования
- Статичные части интерфейса (меню, футеры).
- Элементы, которые не зависят от контекста пользователя.

### Минусы
- Управление только через время жизни (`expires-after`, `expires-sliding`).
- Нельзя вручную очистить кэш.
- Не работает в Blazor.
- Нет зависимости от других данных или событий.

---

## Сводная таблица минусов

| Тип кэша                  | Минусы |
|---------------------------|--------|
| **In-Memory**             | Потеря при перезапуске, не распределён, может утекать память |
| **Redis (распределённый)**| Зависимость от сети, сложнее в настройке, дороже |
| **Response Cache**        | Персонализация сложно, нельзя вручную инвалидировать |
| **Razor Tag Helpers**     | Только по времени, нет зависимостей, не в Blazor |

---

## Best Practices & Tips

### Для In-Memory Cache
- Устанавливайте ограничение размера кэша:
```csharp
builder.Services.AddMemoryCache(options => 
{
    options.SizeLimit = 1024; // условные единицы
});
```
- Инвалидируйте кэш при обновлении данных:
```csharp
_cache.Remove($"product_{id}");
```
- Не используйте `AddOrUpdate` — он не гарантирует потокобезопасности и не предусмотрен в `IMemoryCache`.

### Для Redis
- Настройте политики повторных попыток и таймаутов.
- Логируйте ошибки и реализуйте fallback-логику.
- Используйте мониторинг (RedisInsight, Azure Monitor).

### Для Response Cache
- Избегайте кэширования персональных данных.
- Используйте `Cache-Control: private` если кэшируете клиентские данные.
- Активно применяйте `ETag` и `If-None-Match`.

### Для Tag Helpers
- Не используйте для динамических данных.
- Учитывайте, что они работают только в MVC/View, не в Blazor.

---

## Выводы

Выбор типа кэша зависит от:
- Архитектуры приложения (монолит / микросервисы).
- Нужно ли горизонтальное масштабирование.
- Частоты изменения данных.
- Требований к производительности и отказоустойчивости.
