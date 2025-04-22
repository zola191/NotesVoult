Загрузка данных в EF Core — это процесс выполнения запросов к базе данных для получения сущностей (или данных) из таблиц. Это основной механизм работы с данными в EF Core. Он включает в себя различные стратегии загрузки данных:

- **Explicit Loading (Явная загрузка)** :  
    Данные загружаются явно с помощью методов, таких как `Entry(entity).Collection(...).Load()` или `Entry(entity).Reference(...).Load()`.
    
- **Lazy Loading (Отложенная загрузка)** :  
    Данные загружаются автоматически при первом обращении к навигационным свойствам.
    
- **Eager Loading (Жадная загрузка)** :  
    Связанные данные загружаются вместе с основной сущностью с помощью метода `Include` или `ThenInclude`.
    
- **Raw SQL Queries** :  
    Выполнение произвольных SQL-запросов через методы `FromSqlRaw` или `FromSqlInterpolated`.

```csharp
// Eager Loading
var blogs = context.Blogs
    .Include(b => b.Posts)
    .ToList();

// Explicit Loading
var blog = context.Blogs.Find(1);
context.Entry(blog).Collection(b => b.Posts).Load();
```