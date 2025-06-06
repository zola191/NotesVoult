Compiled Queries (скомпилированные запросы) — это механизм оптимизации, который позволяет компилировать LINQ-запрос один раз и повторно использовать его для выполнения. Это особенно полезно для часто используемых запросов, так как уменьшает накладные расходы на компиляцию LINQ-выражений в SQL.

### Как это работает?

- Вы создаете делегат, который представляет собой скомпилированный запрос.
- Этот делегат можно вызывать многократно, передавая параметры, если они нужны.
- Компиляция происходит только один раз, что улучшает производительность.

```csharp
// Создание скомпилированного запроса
private static readonly Func<MyDbContext, int, Blog> _compiledQuery =
    EF.CompileQuery((MyDbContext context, int id) =>
        context.Blogs
            .Include(b => b.Posts)
            .FirstOrDefault(b => b.Id == id));

// Использование скомпилированного запроса
var blog = _compiledQuery(context, 1);
```

### Особенности:

- Запрос компилируется один раз и может использоваться многократно.
- Уменьшает накладные расходы на анализ LINQ-выражений и генерацию SQL.
- Подходит для часто выполняемых запросов с фиксированной логикой.