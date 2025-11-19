
GitLab — это платформа для DevOps полного цикла: от хранения кода до CI/CD, мониторинга, безопасности и управления инфраструктурой.  
Она объединяет функции GitHub, Jenkins, Jira, Docker Registry, SonarQube, Nexus и частично ArgoCD в одном продукте.

# Основные компоненты GitLab

## 1. Git-репозитории
- хранение кода
- просмотр коммитов, diff, веток и тегов
- Merge Requests (MR)
- code review
- защищённые ветки

## 2. GitLab CI/CD
Пайплайн описывается в файле:
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy
```

CI/CD обеспечивает:
- выполнение заданий через runners
- логи выполнения
- управление артефактами
- автоматические деплои
- поддержку Docker, Kubernetes, Terraform, Ansible и многого другого

## 3. GitLab Runner
Агент для выполнения CI/CD задач.

Типы:
- shared
- project
- self-hosted

Режимы выполнения:
- shell
- Docker
- Docker-in-Docker
- Kubernetes executor

# Управление проектами и командами

## Группы и подгруппы

Используются для организации проектов, управления правами и общими настройками.

## Уровни доступа
- Guest
- Reporter
- Developer
- Maintainer
- Owner
## Issue tracking
Встроенная система управления задачами:
- kanban-доски
- эпики
- задачи
- milestones

# DevOps-функции GitLab
GitLab покрывает весь DevOps-цикл.

## Plan
Issues, epics, roadmaps

## Code
Git, review

## Test
CI/CD, автотесты

## Secure
SAST, DAST, Dependency Scanning

## Deploy
AutoDevOps, деплой в Kubernetes, Helm charts

## Monitor
Интеграция с Prometheus, alerts

# Регистры GitLab

## 1. Container Registry

Хранилище Docker-образов:
```
registry.gitlab.com/namespace/project/app:tag
```

## 2. Package Registry
Поддерживает npm, Maven, PyPI, NuGet, Conan и другие форматы.

# GitLab и Kubernetes

GitLab умеет:
- деплоить в Kubernetes
- управлять Helm-чартами
- использовать GitLab Agent for Kubernetes
- создавать review apps


