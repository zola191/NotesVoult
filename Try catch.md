Проброс исключений

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