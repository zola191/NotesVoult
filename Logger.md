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

```