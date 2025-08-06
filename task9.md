## 🏠 Домашнее задание для курса "Микросервисы на GO"

### Задание 9. Метрики, мониторинг и логирование

Ваш микросервис работает, обрабатывает события, назначает курьеров — пора добавить **наблюдаемость (observability)**: метрики, логирование, мониторинг. Это критически важно для продакшн-среды.

> В этом задании вы научитесь собирать метрики по всем HTTP-ручкам и визуализировать их через Prometheus и Grafana.

---

### ✅ Что нужно сделать

1. **Поднять `Prometheus` и `Grafana` через `docker-compose`**
   - Настроить Prometheus на сбор метрик с вашего сервиса (`/metrics`).
   - Настроить Grafana на подключение к Prometheus как data source.
   - Создать дашборд с базовыми метриками (например: общее количество запросов, по коду ответа, по эндпоинту).

2. **Добавить `prometheus` middleware**
   - Использовать библиотеку [`promhttp`](https://pkg.go.dev/github.com/prometheus/client_golang/prometheus/promhttp) или `go-chi/middleware`.
   - Создать собственный middleware, который:
      - логирует все входящие HTTP-запросы
        - `method` — HTTP-метод запроса (`GET`, `POST`, `PUT`, `DELETE` и т.д.)
        - `path` — путь запроса (`/courier`, `/courier/assign`, и т.д.)
        - `status` — код ответа (`200`, `404`, `500` и т.п.)
        - `duration` — сколько времени заняло выполнение запроса (в миллисекундах или секундах)
      - считает количество запросов по всем api эндпойнтам;
      - Добавить обработчик `GET /metrics`, который отдаёт метрики.

3. **Добавить логирование**
   - Логируйте каждый HTTP-запрос через middleware.
   - Формат лога должен включать:
      - `timestamp` — время запроса
      - `method`, `path`, `status`, `duration`
   - Используйте любую удобную библиотеку логирования (`log`, `zap`, `zerolog`, `slog` и др.).
   - Пример строки лога:
     ```
     [INFO] 2025/08/06 14:00:15 method=POST path=/courier/assign status=200 duration=43ms
     ```
   - Дополнительно можно логировать ошибки, превышения лимитов и события назначения курьера.
   - Убедитесь что в коде нет вызовов fmt.Println, если есть замените на логирование
---

### 📎 Пример используемых метрик

```go
var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "path", "status"},
    )

    httpRequestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "Duration of HTTP requests.",
            Buckets: prometheus.DefBuckets,
        },
        []string{"path"},
    )
)
```