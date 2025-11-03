
**Grafana** — это **платформа для визуализации данных и аналитики** с открытым исходным кодом.  
Она используется для построения **дашбордов** (информационных панелей), мониторинга систем, метрик и бизнес-показателей.

Grafana умеет подключаться к множеству источников данных ([[Prometheus]], InfluxDB, Loki, Elasticsearch, [[MySQL]], PostgreSQL и др.) и отображать их в виде графиков, таблиц и метрик в реальном времени.

##  Основные возможности Grafana

| Возможность   | Описание                                                                     |
| ------------- | ---------------------------------------------------------------------------- |
| Визуализация  | Создание графиков, гистограмм, gauge, heatmap и др.                          |
| Дашборды      | Набор панелей, которые можно группировать по системам, сервисам и окружениям |
| Alerting      | Настройка правил оповещений прямо в Grafana                                  |
| Multi-user    | Разграничение прав доступа, организации и папки                              |
| Плагины       | Расширение функционала (панели, data sources, алертеры и т.д.)               |
| Grafana Cloud | SaaS-версия для мониторинга без развёртывания сервера                        |

##  Архитектура Grafana

```
        ┌────────────────────────┐
        │      Users / Admins     │
        └────────────┬────────────┘
                     │ (HTTP / Web UI)
                     ▼
          ┌────────────────────┐
          │      Grafana        │
          │ - Dashboards        │
          │ - Alerting          │
          │ - Auth              │
          └───────┬─────────────┘
                  │ (Data source API)
        ┌──────────────────────────────┐
        │      Источники данных        │
        │                              │
        │ - Prometheus                 │
        │ - Loki (логи)                │
        │ - Tempo (трейсы)             │
        │ - MySQL, Elasticsearch ...   │
        └──────────────────────────────┘
```

##  Связь Grafana и Prometheus

Prometheus — **собирает и хранит метрики**,  
Grafana — **визуализирует их**.

Grafana не хранит метрики у себя.  
Она обращается к API Prometheus (`/api/v1/query`) и получает нужные данные по запросам, написанным на **PromQL**.

##  Настройка Grafana для работы с Prometheus

### 1. Установка (например, через Docker)

```yaml
version: '3.7'
services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - prometheus
```

### 2. Добавление Prometheus как Data Source

После запуска Grafana:

1. Перейди в **Configuration → Data Sources → Add data source**
2. Выбери **Prometheus**
3. Укажи URL (если через Docker Compose):
    http://prometheus:9090
4. Нажми **Save & Test**

 Если всё ок, Grafana подключится к Prometheus.

### 3. Создание дашборда

Можно:

- создать свой собственный дашборд;
- импортировать готовый с [Grafana Dashboards](https://grafana.com/grafana/dashboards/).

 Пример запроса PromQL в панели:

```promql
rate(http_requests_total[5m])
```

Или:

```promql
avg by (instance) (node_cpu_seconds_total{mode="idle"})
```

### 4. Пример визуализации

|Панель|PromQL запрос|Что показывает|
|---|---|---|
|**CPU Usage**|`100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)`|Загрузка CPU|
|**Memory Usage**|`(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100`|Использование RAM|
|**Disk I/O**|`rate(node_disk_io_time_seconds_total[5m])`|Диск (время операций)|
|**HTTP Requests**|`rate(http_requests_total[1m])`|Скорость запросов к приложению|

##  Алерты в Grafana

С версии **Grafana 8+** в неё встроена собственная система **Alerting** (раньше нужно было использовать Alertmanager Prometheus отдельно).

- Можно создавать алерты прямо в панели (на основе запроса PromQL).
- Настраивать **каналы уведомлений**:
    - Email    
    - Slack   
    - Telegram    
    - Microsoft Teams  
    - Webhook   

Пример визуального правила:

```
Если: avg(rate(http_requests_total[5m])) > 100
в течение: 5m
→ Отправить уведомление в Telegram
```

##  Пример готового дашборда для Prometheus + Node Exporter

Можно импортировать дашборд **ID 1860** (Node Exporter Full) из официального репозитория Grafana:

- [https://grafana.com/grafana/dashboards/1860](https://grafana.com/grafana/dashboards/1860)

Показывает:

- CPU, RAM, Disk, Network
- uptime, load average
- и многое другое
##  Интеграция в Kubernetes

Grafana часто устанавливается в Kubernetes через **Helm Chart**:

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install my-grafana grafana/grafana
```

И связывается с Prometheus, установленным, например, через **kube-prometheus-stack**, который содержит:

- Prometheus
- Alertmanager
- Grafana
- Node Exporter
- kube-state-metrics

##  Преимущества Grafana

 - Поддержка десятков источников данных  
 - Интерактивные дашборды  
 - Гибкие фильтры (templating variables)  
 - Интеграция с Prometheus, Loki, Tempo  
 - Простая настройка оповещений  
 - Open Source и Cloud-версия

##  Недостатки

 Хранит только настройки и дашборды (не метрики)  
 Требует знания PromQL для сложных панелей  
 Без внешней БД (Postgres/MySQL) — хранит настройки локально (SQLite)

##  Связка Prometheus + Grafana (итоговая схема)

```
        ┌─────────────┐
        │ Applications │
        │  /metrics    │
        └──────┬───────┘
               │
               ▼
        ┌─────────────┐
        │ Prometheus  │◄── Exporters (Node, DB, etc)
        │  Storage     │
        └──────┬───────┘
               │ (HTTP API /query)
               ▼
        ┌─────────────┐
        │   Grafana   │
        │ Dashboards  │
        │  Alerts     │
        └──────┬───────┘
               │
               ▼
         Пользователь
```
