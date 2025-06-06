## **Оптимальное логгирование в C#: производительность и детализация**

#### ❌ Плохо (интерполяция):
```csharp
_logger.LogDebug($"User {user.Id} created order {order.Id}"); 
```

**Минусы:**
- Создаёт **новую строку** даже если уровень логгирования `Debug` отключён.
- Нагружает память и CPU (особенно в циклах или при частых вызовах).

#### ✅ Решение (шаблоны с placeholders):
```csharp
_logger.LogDebug("User {UserId} created order {OrderId}", user.Id, order.Id);
```
**Плюсы:**

- Форматирование строки происходит **только если лог записан**.
    
- Совместимо с системами вроде Serilog (структурированное логгирование).

Для операций с высокой стоимостью ошибки (баланс, диагноз) логируйте **всё состояние до/после**.

```csharp
public async Task UpdatePatientAsync(Patient patient, PatientData newData)
{
    var oldData = patient.Clone(); // Глубокое копирование
    
    try
    {
        patient.Update(newData);
        await _db.SaveChangesAsync();
        
        _logger.LogInformation("Patient updated. Old: {@Old}, New: {@New}", 
            oldData, patient); // Сериализует весь объект
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to update patient. Old: {@Old}, Attempted: {@New}", 
            oldData, newData);
        throw;
    }
}
```

**Что даёт:**
- Возможность восстановить состояние до изменения.

**не выполняйте сложные операции (которые тратят CPU/память) для формирования данных лога, если сам лог не будет записан из-за настроек уровня логирования**.

#### **❌ Проблема на примере:**
```csharp
// Плохо: CalculateMetrics() выполнится ВСЕГДА, даже если Debug-логи отключены!
_logger.LogDebug($"Performance metrics: {CalculateMetrics()}");
```
#### **✅ Решение: Проверка уровня логирования**
```csharp
// Хорошо: Проверяем, включён ли Debug, ДО вычислений
if (_logger.IsEnabled(LogLevel.Debug)) 
{
    var metrics = CalculateMetrics(); // Тяжёлая операция
    _logger.LogDebug("Performance metrics: {@Metrics}", metrics);
}
```

### 1. **Используйте уровни логирования правильно**

- **Trace** : Для диагностики низкоуровневых проблем (например, входные данные методов).
- **Debug** : Для разработчиков, чтобы понимать внутреннее состояние приложения.
- **Information** : Для бизнес-логики и важных событий (например, успешная авторизация).
- **Warning** : Для потенциально опасных ситуаций (например, повторная попытка подключения).
- **Error** : Для ошибок, которые не должны происходить в нормальной работе.
- **Critical** : Для критических ошибок, требующих немедленного вмешательства.

### **Структурированное логгирование**
- Используйте `{@Object}` для сериализации объектов. Это позволяет анализировать логи программно.
```csharp
_logger.LogInformation("Order processed: {@Order}", order);
```

- Логируйте только то, что действительно нужно для анализа проблем.
- Избегайте дублирования логов