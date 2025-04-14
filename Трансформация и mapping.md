Mapping данных в ASP.NET (и ASP.NET Core) — это процесс преобразования данных между различными слоями приложения, такими как **DTO (Data Transfer Objects)** , **ViewModels** , **Domain Models** и **Entity Models** . Это необходимо для того, чтобы:

1. **Изолировать слои приложения** : Например, скрыть внутреннюю логику бизнес-логики от клиентов API.
2. **Упростить передачу данных** : DTO часто используются для передачи данных через API, чтобы избежать лишних полей или циклических ссылок.
3. **Защитить чувствительные данные** : Исключить поля, которые не должны быть видны клиентам (например, пароли или внутренние идентификаторы).
4. **Совместимость с клиентами** : Преобразование данных под формат, который ожидает клиент.

## Как происходит mapping?

### 1.1 **Ручной маппинг**
#### Когда использовать:
- Для простых моделей.
- В высоконагруженных системах, где важно минимизировать накладные расходы.
- Когда нужно полный контроль над процессом маппинга.

```csharp
// Внутри DTO
public static CategoryDto ToDto(Category entity) => new() 
{
    Id = entity.Id,
    Name = entity.Name.Trim()
};
```

### **1.2 Record Types (C# 9+)**
**Идеально для:**
- Неизменяемых моделей
- Value-объектов


```csharp
public record CategoryDto(Guid Id, string Name, string Description);

// Использование:
var dto = new CategoryDto(entity.Id, entity.Name, entity.Description);
```


## **2. Продвинутые техники**

### **2.1 Паттерн Builder**

```csharp
public class Category // Entity
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
}

public class CategoryDto // DTO
{
    public Guid Id { get; private set; }
    public string Name { get; private set; }
    public string Description { get; private set; }

    // Приватный конструктор
    private CategoryDto() {} 

    // Builder
    public class Builder 
    {
        private readonly CategoryDto _dto = new();

        public Builder WithId(Guid id) 
        {
            _dto.Id = id;
            return this;
        }

        public Builder WithName(string name) 
        {
            _dto.Name = name ?? throw new ArgumentNullException(nameof(name));
            return this;
        }

        public Builder WithDescription(string description) 
        {
            _dto.Description = description;
            return this;
        }

        public CategoryDto Build() 
        {
            if (string.IsNullOrEmpty(_dto.Name))
                throw new InvalidOperationException("Name is required");

            return _dto;
        }
    }
}
```

## 3. **Использование AutoMapper**
AutoMapper — популярная библиотека, которая автоматизирует процесс маппинга. Она особенно удобна для сложных моделей с вложенными объектами.
#### Когда использовать:
- Для сложных моделей с большим количеством полей.
- Когда нужно быстро настроить маппинг.
- Для проектов, где важна скорость разработки, а производительность не критична.
AutoMapper стал платным и использует под капотом много рефлексии, что сказывается на производительности, рефлексия может поменять даже приватные поля

## Где происходит mapping в ASP.NET Core?

Mapping данных обычно выполняется в следующих местах:

1. **Контроллеры** :
    - Преобразование входящих данных (DTO) в модели для бизнес-логики.
    - Преобразование результатов работы бизнес-логики в формат, который будет отправлен клиенту.

2. **Сервисы** :
    - Бизнес-логика может требовать преобразования данных между уровнями (например, из DTO в Domain Model).
3. **Репозитории** :
    - Преобразование Entity Models в DTO или Domain Models.
4. **Middleware** :
    - Иногда маппинг используется для обработки запросов/ответов на уровне middleware.

Mapping из Domain Model в ViewModel **может выполняться в контроллере** , но это не рекомендуется для больших и сложных проектов. Лучше вынести маппинг в сервисы, отдельные классы или даже репозитории. Это сделает код более структурированным, тестируемым и поддерживаемым.


## **Антипаттерны**

❌ **God DTO** - один DTO для всех сценариев  
❌ **Двунаправленный маппинг** - разные пути для Model→DTO и DTO→Model  
❌ **Рефлексия** - медленно и небезопасно

## **Чеклист выбора стратегии**
1. **До 3 полей** → Record types
2. **4-10 полей** → Ручной маппинг
3. **Сложная логика** → Builder
4. **Частые изменения** → AutoMapper
5. **High-load** → Оптимизированный ручной маппинг