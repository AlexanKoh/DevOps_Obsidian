
**Terraform** — это **инфраструктура как код (IaC)**, созданная компанией **HashiCorp**.  
Он позволяет **описывать, создавать, изменять и уничтожать** инфраструктуру (серверы, сети, базы данных, Kubernetes-кластеры и т.д.) с помощью **декларативных конфигураций** на языке **HCL (HashiCorp Configuration Language)**.

Ты **описываешь желаемое состояние** инфраструктуры в `.tf` файлах —  
а Terraform сам вычисляет, **что нужно создать, изменить или удалить**, чтобы привести реальную инфраструктуру к этому состоянию.

Пример:

```hcl
provider "aws" {
  region = "eu-west-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "my-web-server"
  }
}
```

Командой:

```bash
terraform apply
```

Terraform создаст EC2-инстанс в AWS.

##  Архитектура Terraform

### 1. **Provider (провайдер)**

Плагин, который знает, как взаимодействовать с конкретным сервисом.  
Например:

- `aws` — Amazon Web Services
- `google` — Google Cloud Platform
- `azurerm` — Microsoft Azure
- `kubernetes` — для работы с кластерами
- `helm` — для установки Helm-чартов
- `null`, `local`, `random` — встроенные служебные провайдеры

### 2. **Resource (ресурс)**

Основной строительный блок: виртуальная машина, сеть, база, кластер и т.п.

### 3. **Data source**

Позволяет **читать существующие данные** (например, существующую сеть VPC или AMI-образ).

### 4. **Module (модуль)**

Набор ресурсов, объединённых в переиспользуемый компонент.  
Ты можешь использовать:

- свои локальные модули (`./modules/...`)
- или публичные (`registry.terraform.io`).

### 5. **State (состояние)**

Terraform хранит файл `terraform.tfstate`, где записывает текущее состояние инфраструктуры.  
Этот файл **очень важен**: он нужен, чтобы Terraform понимал, что уже создано, и что нужно изменить.

Можно хранить:

- **локально** — `terraform.tfstate`
- **удалённо** — в **S3**, **Terraform Cloud**, **GCS**, **Azure Blob** и т.д.

##  Основные команды Terraform

|Команда|Описание|
|---|---|
|`terraform init`|Инициализация (загрузка провайдеров, модулей)|
|`terraform plan`|Показывает, что будет создано/изменено/удалено|
|`terraform apply`|Применяет изменения|
|`terraform destroy`|Уничтожает созданные ресурсы|
|`terraform validate`|Проверяет синтаксис|
|`terraform fmt`|Форматирует `.tf` файлы|
|`terraform output`|Показывает выходные значения (outputs)|
|`terraform import`|Импортирует уже существующие ресурсы в стейт|
|`terraform state`|Управление состоянием (просмотр, удаление и т.д.)|

##  Основные концепции HCL (Terraform Language)

### 🔸 Переменные

```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

Использование:

```hcl
instance_type = var.instance_type
```

### 🔸 Выходные данные (outputs)

```hcl
output "public_ip" {
  value = aws_instance.web.public_ip
}
```

### 🔸 Локальные значения (locals)

```hcl
locals {
  env_prefix = "prod"
}

resource "aws_s3_bucket" "bucket" {
  bucket = "${local.env_prefix}-my-bucket"
}
```

### 🔸 Зависимости (depends_on)

Позволяет явно указать порядок создания ресурсов:

```hcl
resource "aws_instance" "web" {
  depends_on = [aws_security_group.web_sg]
}
```

##  Terraform в DevOps

Terraform часто используется совместно с другими инструментами:

- **[[Packer]]** — создание образов (AMI, Docker, и т.д.)
- **[[Ansible]] / [[Chef]] / [[Puppet]]** — настройка ПО внутри машин
- **[[Kubernetes]] + [[Helm]]** — для управления кластерами и чартами
- **[[Jenkins]] / [[GitLab]] CI** — автоматизация инфраструктурных пайплайнов
- **[[Vault]]** — хранение секретов
- **[[Terragrunt]]** — надстройка для организации больших проектов Terraform

##  Примеры использования

- Развёртывание инфраструктуры AWS (VPC, EC2, RDS, S3)
- Создание GKE кластера в GCP
- Создание Azure Resource Group и Kubernetes Service
- Настройка Kubernetes объектов (`kubernetes_deployment`, `kubernetes_service`)
- Управление DNS-записями (Cloudflare, Route53)
- Автоматизация CI/CD-инфраструктуры

##  Типичный рабочий процесс

1.  Написать `.tf` конфигурацию
2. `terraform init`
3.  `terraform plan`
4.  `terraform apply`
5.  Проверить `terraform output`
6. `terraform destroy` (если нужно всё снести)

##  Пример: AWS + Terraform

```hcl
provider "aws" {
  region = "eu-central-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "subnet" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "eu-central-1a"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.subnet.id

  tags = {
    Name = "web-server"
  }
}

output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

##  Лучшие практики

✅ Используй **модули** — не дублируй код.  
✅ Храни `terraform.tfstate` **удалённо** (например, в S3).  
✅ Применяй **локальные и глобальные переменные** (`locals`, `variables.tf`).  
✅ Делай **plan → apply** через CI/CD.  
✅ Настраивай **backend с блокировкой** (например, DynamoDB для AWS).  
✅ Разделяй окружения: `dev`, `staging`, `prod`.  
✅ Используй **workspaces** или отдельные каталоги для окружений.

##  Частые ошибки

- 🔴 Удалён `terraform.tfstate` → Terraform «забывает» про ресурсы
- 🔴 Несогласованные версии провайдеров
- 🔴 Изменение ресурсов вручную в облаке → Terraform конфликтует
- 🔴 Конфигурация без бэкенда → потеря стейта при CI
- 🔴 Секреты в `.tf` файлах вместо переменных окружения

##  Terraform Cloud / Enterprise

HashiCorp предоставляет SaaS-сервис — **Terraform Cloud**,  
который хранит состояние, запускает `plan/apply` в облаке и управляет правами доступа.  
Удобно для командной работы и интеграции с GitHub/GitLab.

## 🧩 Альтернативы

|Инструмент|Особенности|
|---|---|
|**Pulumi**|Использует языки (Python, Go, TS) вместо HCL|
|**CloudFormation**|Только для AWS|
|**Ansible**|Императивный подход (отличие от декларативного Terraform)|
|**Crossplane**|IaC внутри Kubernetes|
|**CDK for Terraform (CDKTF)**|Пишешь Terraform через TypeScript/Python|
