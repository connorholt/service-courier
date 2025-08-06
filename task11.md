## 🏠 Домашнее задание для курса "Микросервисы на GO"

### Задание 11. Линтинг, проверка стиля и CI-пайплайн

Чистый и единообразный код — это признак зрелого проекта. На этом этапе ты добавишь **автоматическую проверку кода** с помощью линтеров и настроишь **CI-пайплайн**, который будет запускать линтер и тесты при каждом коммите и пуше.

> Это основа для командной работы и подготовки проекта к продакшену.

---

### ✅ Что нужно сделать

1. **Добавить линтеры в проект**
    - Используй [`golangci-lint`](https://golangci-lint.run/) — один из самых популярных инструментов в Go.
    - Установи линтер локально или через Docker.
    - Добавь конфигурационный файл `.golangci.yml` в корень проекта. Пример базовой конфигурации:
      ```yaml
      run:
        timeout: 3m
      linters:
        enable:
          - govet
          - staticcheck
          - gofmt
          - goimports
          - errcheck
          - unused
          - gocritic
      issues:
        exclude-use-default: false
      ```

2. **Настроить GitHub Actions**
    - Добавь workflow-файл `.github/workflows/ci.yml`, который:
        - устанавливает Go;
        - запускает линтер;
        - запускает тесты;
        - может проверять покрытие.
    - Пример содержимого:

      ```yaml
      name: Go CI
 
      on:
        push:
          branches: [ main ]
        pull_request:
          branches: [ main ]
 
      jobs:
        build:
 
          runs-on: ubuntu-latest
 
          steps:
          - name: Checkout code
            uses: actions/checkout@v3
 
          - name: Set up Go
            uses: actions/setup-go@v4
            with:
              go-version: 1.21
 
          - name: Install golangci-lint
            run: |
              curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.55.2
 
          - name: Run golangci-lint
            run: golangci-lint run ./...
 
          - name: Run tests
            run: go test ./... -v
      ```

3. **Убедиться, что пайплайн работает**
    - Сделай тестовый коммит — GitHub должен запустить CI.
    - Убедись, что линтер и тесты проходят.
    - Попробуй специально написать ошибку (например, неиспользованную переменную) — линтер должен упасть.

