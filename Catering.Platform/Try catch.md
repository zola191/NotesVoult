```csharp
try
{
    // Код, который может выбросить исключение
    var result = await _someService.DoSomethingAsync();
}
catch (SpecificException ex) // Ловим конкретное исключение
{
    _logger.LogError(ex, "Ошибка в DoSomethingAsync");
    throw; // Пробрасываем дальше (со стектрейсом)
}
catch (Exception ex) // Ловим все остальные исключения
{
    _logger.LogError(ex, "Неизвестная ошибка");
    throw new CustomException("Произошла ошибка", ex); // Оборачиваем в кастомное исключение
}
finally
{
    // Код, который выполнится в любом случае (например, освобождение ресурсов)
}
```


| Вариант                 | Сохраняет стек вызовов?                 | Когда использовать                             |
|-------------------------|-----------------------------------------|------------------------------------------------|
| throw;                  | ✅ Да                                    | Если нужно просто пробросить исключение дальше |
| throw ex;               | ❌ Нет (перезаписывает стектрейс)        | Почти никогда (теряет контекст ошибки)         |
| throw new CustomEx(ex); | ✅ (если передать ex как innerException) | Если нужно обернуть исключение в кастомный тип |
- **Ловить** → если можно обработать ошибку на месте (например, вернуть `404` при отсутствии записи).
    
- **Пробрасывать** → если ошибка критическая и её должна обработать глобальная middleware.

В ASP.NET Core можно перехватывать все необработанные исключения через **Middleware**:

```csharp
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var exceptionHandler = context.Features.Get<IExceptionHandlerFeature>();
        var exception = exceptionHandler?.Error;
        
        _logger.LogError(exception, "Global exception");
        
        context.Response.StatusCode = exception switch
        {
            NotFoundException => 404,
            _ => 500
        };
        
        await context.Response.WriteAsJsonAsync(new { error = exception?.Message });
    });
});
```

В асинхронных методах исключения **не теряются**, но их нужно правильно ловить:
Исключение выбросится **только при await**, а не при вызове метода.

```csharp
try
{
    await SomeAsyncMethod();
}
catch (Exception ex) // Сработает, даже если исключение было в асинхронном коде
{
    _logger.LogError(ex, "Async error");
    throw;
}
```

Если вызвать их в синхронном коде — исключение оборачивается в `AggregateException`.

```csharp
// Плохо в синхронном коде:
try 
{
    SomeAsyncMethod().Wait(); // Исключение будет AggregateException!
}
catch (AggregateException ex) 
{
    var innerEx = ex.InnerException; // Настоящая ошибка внутри
}
```

итого нужно использовать `async/await` везде, где можно.

Можно создавать свои типы исключений для разных сценариев:
```csharp
public class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message) { }
}

// Использование:
throw new NotFoundException("User not found");
```

✅ **Ловите только те исключения, которые можете обработать.**  
✅ **Используйте `throw;` вместо `throw ex;` (если не нужно оборачивать).**  
✅ **Логируйте ошибки (`ILogger` + Sentry/ELK).**  
✅ **Глобальная обработка через Middleware.**  
✅ **Не игнорируйте исключения (даже в `catch`).**  
✅ **Используйте кастомные исключения для бизнес-ошибок.**