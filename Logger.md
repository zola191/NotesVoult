## **Оптимальное логгирование в C#: производительность и детализация**

#### ❌ Плохо (интерполяция):
```csharp
_logger.LogDebug($"User {user.Id} created order {order.Id}"); 
```