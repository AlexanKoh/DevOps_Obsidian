
#  Что такое Helm и Helm Charts

**Helm** — это пакетный менеджер для Kubernetes, аналогичный apt/yum для Linux или npm/pip для разработчиков.  
Он позволяет **упаковывать**, **версионировать**, **переиспользовать** и **деплоить** приложения в Kubernetes.

**Helm Chart** — это «пакет» для установки приложения в Kubernetes.  
Он содержит шаблоны YAML-манифестов, параметры и структуру для гибкой конфигурации.

#  Зачем нужны Helm Charts

Без Helm приходится писать множество YAML-файлов вручную:
- Deployment
- Service
- ConfigMap
- Secret
- PVC
- Ingress
- и т.д.

Helm решает проблемы:
- DRY (не повторять код)
- параметризация (values.yaml)
- версионирование (Chart.yaml)
- идемпотентное обновление (helm upgrade)
- удобное удаление и rollback

#  Структура Helm Chart

Когда вы создаёте chart:
```
helm create mychart
```

Будет создано:
```
mychart/
  Chart.yaml             # Метаданные chart’a
  values.yaml            # Значения по умолчанию
  templates/             # Шаблоны Kubernetes-манифестов
  charts/                # Зависимости
  .helmignore            # Игнорируемые файлы
```

#  Главные файлы и их роль

## **Chart.yaml**

Определяет:
- имя чартa
- версию
- описание
- зависимости
Пример:
```yaml
apiVersion: v2
name: myapp
version: 0.1.0
description: My application
appVersion: "1.0.0"
```

## **values.yaml**

Значения по умолчанию, которые можно переопределять:
```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
```

## **templates/**

Файлы с расширениями `.yaml`, но внутри могут содержать Go-шаблоны:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
```

#  Шаблонизатор Helm (Go templates)

Helm использует мощный templating engine:
- `{{ .Values.* }}` — доступ к значениям
- `{{ .Chart.* }}` — доступ к данным чартa
- `{{ include }}` и `{{ template }}` — переиспользование шаблонов
- `{{ toYaml }}` — автоматическая генерация YAML
- условные конструкции `if/else`
- циклы `range`

Пример:
```yaml
{{- if .Values.ingress.enabled }}
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "myapp.fullname" . }}
{{- end }}
```

#  Установка и использование Helm Charts

### Установка chart:
```
helm install myapp ./mychart
```

### Обновление:
```
helm upgrade myapp ./mychart
```

### Удаление:
```
helm uninstall myapp
```

### Dry-run (прогон без применения):
```
helm install myapp ./mychart --dry-run --debug
```

#  Rollback

Helm хранит историю релизов:
```
helm history myapp
```

Откат:
```
helm rollback myapp 1
```

#  Переопределение values

При установке можно подставить свои значения:
```
helm install myapp ./mychart -f custom-values.yaml
```

или:
```
helm install myapp ./mychart --set replicaCount=5
```

#  Зависимости Charts

Helm может подтягивать другие charts.

Пример в `Chart.yaml`:
```yaml
dependencies:
  - name: mysql
    version: 1.2.3
    repository: "https://charts.bitnami.com/bitnami"
```

Загрузить зависимости:
```
helm dependency update
```

#  Репозитории Helm

Добавить репозиторий:
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Список:
```
helm repo list
```

Поиск:
```
helm search repo mysql
```

#  Лучшие практики создания chart’ов

### 1. Не жёстко прописывать параметры

— всё должно быть в values.yaml
### 2. Использовать helpers.txt

В файле `templates/_helpers.tpl` определяются функции, например:
```yaml
{{- define "myapp.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end }}
```

### 3. Использовать аннотации и liveness/readiness probes
### 4. Делить values на маленькие части

Например:
- values-production.yaml
- values-staging.yaml

### 5. lint перед деплоем
```
helm lint mychart
```

