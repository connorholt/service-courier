## 🏠 Домашнее задание для курса "Микросервисы на GO"

### Задание 4. Назначение и снятие курьера с заказа
Теперь наш сервис становится умнее — пора научиться **назначать курьеров на заказы** и **отслеживать доставку во времени**.  
Но самое главное — всё это должно быть реализовано **архитектурно правильно**:
- бизнес-логика вынесена в отдельные usecase-компоненты
- ответственность за работу с типом транспорта — выделена в отдельную **фабрику**
- данные сохранены в базе в отдельной таблице `courier_orders`

Мы продолжаем проектировать систему, которую легко поддерживать и расширять.

#### ✅ Что нужно сделать

1. Добавить две новые API-ручки:
   - `POST /courier/assign` — назначение курьера на заказ
   - `POST /courier/unassign` — снятие курьера с заказа

2. Расширить таблицу `couriers`:
   - новое поле `transport_type` (`on_foot`, `scooter`, `car`)
   - по умолчанию — `on_foot`

3. Создать новую таблицу `courier_orders`:
   - связь курьера и заказа
   - время назначения
   - ожидаемое время доставки (delivery_deadline)

4. Добавить фабрику `DeliveryTimeFactory`, которая определяет дедлайн по типу транспорта (пока это будет использоваться только при выводе):
   - `on_foot` — +30 минут
   - `scooter` — +15 минут
   - `car` — +5 минут

5. Реализовать бизнес-логику назначения/снятия через usecase слой (или другой слой, в зависимости от архитектуры).

6. Обновить ответ в `GET /couriers`:
   - должен включать `transport_type` для каждого курьера.

## 📄 Таблица `couriers`

```sql
CREATE TABLE couriers (
    id              SERIAL PRIMARY KEY,
    name            TEXT NOT NULL,
    status          TEXT NOT NULL DEFAULT 'available',
    transport_type  TEXT NOT NULL DEFAULT 'on_foot'  -- on_foot | scooter | car
);
```

```sql
CREATE TABLE courier_orders (
id                  SERIAL PRIMARY KEY,
courier_id          INTEGER NOT NULL REFERENCES couriers(id),
order_id            INTEGER NOT NULL,
assigned_at         TIMESTAMP NOT NULL DEFAULT NOW(),
delivery_deadline   TIMESTAMP NOT NULL
);
```

### 🌐 **4. API: `POST /courier/assign`**

4.1 🔁 POST /courier/assign Назначить курьера на заказ. Кандидат выбирается автоматически* из списка `available`.

**Тело запроса:**
```json
{
  "order_id": 123,
  "order_created_at": "2025-08-06T13:00:00Z"
}
```
**Ответ:**
```json
{
  "courier_id": 5,
  "order_id": 123,
  "transport_type": "scooter",
  "delivery_deadline": "2025-08-06T13:15:00Z"
}
```
**Коды ответов:**
- `200 OK` — курьер назначен
- `409 Conflict` — нет свободных курьеров
- `400 Bad Request` — отсутствуют необходимые поля
* Логика выбора пока не важна, остается на Ваше усмотрение.

4.2 🔁 POST /courier/unassign  Снять курьера с заказа. Возвращает курьера в статус `available`.

**Тело запроса:**
```json
{
  "order_id": 123
}
```

**Ответ:**
```json
{
"order_id": 123,
"status": "unassigned"
}
```

**Коды ответов:**
- `200 OK` — курьер снят
- `404 Not Found` — связь курьера и заказа не найдена
- `400 Bad Request` — невалидные данные

4.3 🔍 GET /couriers  Возвращает список всех курьеров.

**Ответ:**
```json
[
  {
    "id": 1,
    "name": "Ivan",
    "status": "available",
    "transport_type": "on_foot"
  },
  {
    "id": 2,
    "name": "Olga",
    "status": "busy",
    "transport_type": "car"
  }
]
```

#### 🧩 **7. Архитектура: usecase и фабрика**

- **Контроллеры (handler)** — принимают запрос и отдают ответ
- **Usecase (в `internal/service`)** — бизнес-логика назначения и снятия
- **Фабрика (`pkg/deliverytime`)** — вычисляет дедлайн доставки по типу транспорта
- **Репозитории (`internal/repository`)** — работа с базой