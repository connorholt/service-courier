## 🏠 Домашнее задание для курса "Микросервисы на GO"

### Задание 10. Rate Limiter и Retry при обращении к внешнему сервису

Когда сервис работает под нагрузкой или вызывает сторонние API — важно уметь **ограничивать частоту запросов (rate limiting)**, чтобы не перегружать себя и других. А при получении `429 Too Many Requests` — **повторять запрос (retry)**, а не сразу падать с ошибкой.

> В этом задании ты добавишь rate limiter к своему сервису и научишься обрабатывать ограничения при интеграции с другим микросервисом (`service-order`).

---

### ✅ Что нужно сделать

1. **Добавить Rate Limiter в `service-courier`**
    - Ограничить количество входящих запросов к API (например, 5 RPS на IP или глобально).
    - Можно использовать готовые библиотеки (`golang.org/x/time/rate`, `github.com/ulule/limiter`, `github.com/didip/tollbooth`) или реализовать свою.
    - Подключить rate limiter как middleware.
    - При превышении лимита возвращать `429 Too Many Requests`.

2. **Обновить `service-order` gateway**
    - При получении ответа `429` (или временных ошибок) от `service-courier` — **повторить запрос с задержкой (retry)**.
    - Реализовать **экспоненциальную задержку** (backoff) или фиксированный интервал между попытками (например, 100–200 мс).
    - Максимум 3–5 попыток.

3. **Добавить логирование и метрики**
    - Логировать каждый случай превышения лимита (`rate limit exceeded`).
    - Добавить счётчик Prometheus:
        - `rate_limit_exceeded_total`
        - `gateway_retries_total`

4. **Обновить тесты**
    - Покрыть retry-логику.
    - Проверить поведение при превышении лимита.
    - При желании — протестировать собственную реализацию rate limiter'а.

---

### 🔧 Пример кода (RateLimiter middleware)

```go
limiter := rate.NewLimiter(5, 10) // 5 RPS, burst up to 10

func RateLimitMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !limiter.Allow() {
            http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
            return
        }
        next.ServeHTTP(w, r)
    })
}
