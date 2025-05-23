    // лучше public static readonly т.к. consts меняется только при повторной компиляции всего solution и зависимых проектов
    // public static readonly подставит ссылку на объект, const быстрее чем readonly выбор зависит от контекста
    // const можно использовать внтури сборки или приватные внутри класса

### 1. **`const`**

- **Определение** :  
    Константа (`const`) — это неизменяемое значение, которое определяется во время компиляции.
    
- **Характеристики** :
    
    - Значение должно быть известно на этапе компиляции.
    - Может быть примитивным типом (например, `int`, `double`, `string`) или `null`.
    - Значение внедряется напрямую в код при компиляции (инлайнится).
    - Если константа изменяется, требуется перекомпиляция всего решения и зависимых проектов.
    - Используется внутри сборки или как приватные члены класса.

```csharp
public class Constants
{
    public const int MaxValue = 100;
    public const string AppName = "MyApp";
}

// Использование
Console.WriteLine(Constants.MaxValue); // Внедряется значение 100 напрямую
```

- **Плюсы** :
    
    - Быстрее, так как значение известно на этапе компиляции и инлайнится.
    - Простота использования для фиксированных значений.
- **Минусы** :
    
    - Нет гибкости: если значение изменяется, требуется перекомпиляция всех зависимых проектов.
    - Ограничено примитивными типами.

### 2. **`readonly`**

- **Определение** :  
    Поле `readonly` — это неизменяемое значение, которое может быть задано либо при инициализации, либо в конструкторе.
    
- **Характеристики** :
    
    - Значение определяется во время выполнения (runtime).
    - Может быть любого типа, включая ссылочные типы (например, объекты, массивы).
    - Не инлайнится, а хранится как ссылка на объект.
    - Изменение значения возможно только в конструкторе.

```csharp
public class Configuration
{
    public static readonly int MaxValue = 100;
    public static readonly string AppName = "MyApp";

    public Configuration(int maxValue)
    {
        MaxValue = maxValue; // Можно изменить только в конструкторе
    }
}

// Использование
Console.WriteLine(Configuration.MaxValue); // Выводит значение через ссылку
```

- **Плюсы** :
    
    - Гибкость: можно задавать значения во время выполнения.
    - Поддерживает любые типы данных, включая сложные объекты.
    - Не требует перекомпиляции зависимых проектов при изменении значения.
- **Минусы** :
    
    - Медленнее, чем `const`, так как значение не инлайнится.

### 3. **Когда использовать?**

#### a) **Использование `const`**

- Когда значение:
    - Известно на этапе компиляции.
    - Не изменяется никогда (например, математические константы, такие как `PI`).
    - Примитивного типа.

#### b) **Использование `readonly`**

- Когда значение:
    - Должно быть вычислено во время выполнения.
    - Может зависеть от входных данных или конфигурации.
    - Сложного типа (например, объекты, массивы).