# Идемпотентность в распределённых системах

**Идемпоте́нтность** (от лат. *idem* — «тот же самый» + *potens* — «способный»)  
— это свойство операции, при котором повторный вызов даёт тот же результат, что и первый.

---

## 🔥 Зачем нужна идемпотентность

1. **Повторные запросы безопасны**  
   Сеть ненадёжна: запрос может потеряться, таймаутнуться или выполниться дважды.  
   Если операция идемпотентна → повтор не портит данные.

   **Пример:**  
   `PUT /user/1 { "name": "Sergo" }`  
   Сколько раз не отправь → имя всегда будет `"Sergo"`.

---

2. **Упрощает retry-механику**  
   Клиент/сервис может ретраить запросы при ошибках без риска дублирования.  
   В интеграциях через Kafka или Celery сообщение может прийти дважды — идемпотентный обработчик гарантирует корректное состояние.

---

3. **Согласованность данных**  
   Даже при сбое или повторном вызове результат всегда одинаковый → данные остаются консистентными.

---

4. **Простота тестирования**  
   Можно легко проверить поведение: вызов 1, 2 или 10 раз → результат одинаков.

---

5. **Безопасность API (REST)**  
   В спецификации HTTP:
   - `GET`, `PUT`, `DELETE` → **идемпотентные**
   - `POST` → **неидемпотентный** (каждый вызов создаёт новый ресурс)

---

## ⚙️ Реализация идемпотентности в FastAPI

Используем Redis для хранения результатов по `Idempotency-Key`.

```python
from fastapi import APIRouter, Request
from redis.asyncio import Redis
import json

redis = Redis(host="localhost", port=6379, db=0)

class IdempotentAPIRouter(APIRouter):
    def add_api_route(self, path: str, endpoint, **kwargs):
        async def wrapper(request: Request, *args, **kw):
            key = request.headers.get("Idempotency-Key")
            if not key:
                # Если ключ не передан → обычное выполнение
                return await endpoint(request, *args, **kw)

            # Проверяем кэш
            cached = await redis.get(key)
            if cached:
                return json.loads(cached)

            # Выполняем запрос
            result = await endpoint(request, *args, **kw)

            # Сохраняем результат на 1 час
            await redis.setex(key, 3600, json.dumps(result))
            return result

        super().add_api_route(path, wrapper, **kwargs)


router = IdempotentAPIRouter()

@router.post("/pay")
async def pay(request: Request):
    body = await request.json()
    return {"order": body["orderId"], "status": "paid"}

```

✅ Преимущества подхода

Единая точка контроля (DRY).

Легкая настройка TTL и правил кэширования.

Работает во всех сервисах → можно переиспользовать.

Возможность метрик: считать количество повторных запросов.
