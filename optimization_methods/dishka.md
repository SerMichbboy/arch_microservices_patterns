# 🧩 Dishka Manifest: Рекомендации по внедрению DI-контейнера в Python-проект

## 📘 Общее описание

**Dishka** — это лёгкий и современный Dependency Injection (DI) контейнер для Python, созданный для управления зависимостями в асинхронных и многослойных приложениях. Он помогает избавиться от ручной передачи зависимостей, упрощает архитектуру и делает код более читаемым и тестируемым.

Dishka особенно хорошо работает с фреймворками:
- FastAPI
- aiohttp
- Telegram bots

---

## 🎯 Цели библиотеки

- ✅ Централизация и автоматизация создания зависимостей
- ✅ Поддержка асинхронных объектов и их финализация
- ✅ Упрощение тестирования через подмену зависимостей
- ✅ Чистая интеграция с современными Python-фреймворками
- ✅ Снижение связности между слоями приложения

---

## 🛠 Рекомендации по внедрению

### 📦 Где размещать контейнер

Создай файл `infrastructure/di.py` — это будет точка входа для всех зависимостей:

```python
from dishka import Provider, Scope
from infrastructure.db.mongo import get_mongo_client
from infrastructure.kafka.producer import get_kafka_producer

provider = Provider(scope=Scope.APP)
provider.provide(get_mongo_client)
provider.provide(get_kafka_producer)
```

## 🧠 Как регистрировать зависимости
Функции, которые создают объекты, должны быть асинхронными и возвращать готовые экземпляры:

Функции, которые создают объекты, должны быть асинхронными и возвращать готовые экземпляры:

```python
# infrastructure/db/mongo.py
from motor.motor_asyncio import AsyncIOMotorClient

async def get_mongo_client() -> AsyncIOMotorClient:
    return AsyncIOMotorClient("mongodb://localhost:27017")
```


## 🧱 Архитектурный шаблон (TDD + Clean Architecture)


  src/
├── api/                # Входные точки: роуты, сериализация
│   ├── v1/
│   └── routers/
│   └── schemas/
│
├── application/        # Use-cases, бизнес-логика
│   ├── services/
│   └── interfaces/
│
├── domain/             # Сущности, value objects
│   ├── models/
│   └── schemas/
│
├── infrastructure/     # Реализация интерфейсов: БД, Kafka, Redis
│   ├── db/
│   ├── kafka/
│   ├── config.py
│   └── di.py

## 💡 Примеры лучших практик
### 🔧 Инъекция в сервис


```python
from dishka import inject, FromDishka
from motor.motor_asyncio import AsyncIOMotorClient
from aiokafka import AIOKafkaProducer

@inject
class UserService:
    def __init__(
        self,
        db: AsyncIOMotorClient = FromDishka(),
        kafka: AIOKafkaProducer = FromDishka()
    ):
        self.db = db
        self.kafka = kafka

```

### 🚀 Интеграция с FastAPI


```python
from fastapi import FastAPI
from dishka.integrations.fastapi import setup_dishka, DishkaRoute
from infrastructure.di import provider

app = FastAPI()
setup_dishka(app, provider=provider, route_class=DishkaRoute)

```

### 🧪 Тестирование с подменой зависимостей

```python
from dishka import Provider

test_provider = Provider()
test_provider.provide(lambda: MockKafkaProducer())
test_provider.provide(lambda: FakeMongoClient())

# Используй test_provider в тестовом FastAPI-приложении
```

## ✅ Заключение
Dishka — это не просто DI-контейнер, это инструмент архитектурной дисциплины. Он помогает строить приложения, где зависимости управляются централизованно, код становится чище, а тесты — проще. Внедряй Dishka с самого начала, и ты избежишь множества архитектурных ловушек.

      
