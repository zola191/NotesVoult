#CSharpInterviewFlashcards

## 1. Что такое middleware
?
это часть пайплайна (конвейера) запроса. Запрос, приходя на наш бекенд, по-очереди обрабатывается различными middleware прежде чем дойдет до контроллера и после его (контроллера) отработки.

Middleware бывают: для логгирования, для авторизации, роутинга, измерения скорости запроса. Они могут получить доступ к HttpContext и обработать или обогатить запрос дополнительными данными, прежде, чем он дойдет до контроллера.

### 1. **Определение Middleware** :
```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine("Logging: Request received.");
        await _next(context); // Передача управления следующему middleware
        Console.WriteLine("Logging: Response sent.");
    }
}
```
### 2. **Регистрация Middleware** :
```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        app.UseMiddleware<LoggingMiddleware>();
        app.Run(async context =>
        {
            await context.Response.WriteAsync("Hello, World!");
        });
    }
}
```

## Пример Inline Middleware

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Before handling request");
    await next(); // Передаем управление следующему middleware
    Console.WriteLine("After handling request");
});
```