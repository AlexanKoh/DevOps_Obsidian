
#  **Что такое Vagrant**

**Vagrant** — это инструмент для создания и управления виртуальными машинами через простой конфигурационный файл (**Vagrantfile**).

Он позволяет разработчикам быстро и одинаково поднимать окружения на базе:
- VirtualBox
- VMware
- Hyper-V
- Docker
- Libvirt/KVM
- AWS и др.

Главная цель — **создавать воспроизводимые окружения**, чтобы у всех разработчиков была одинаковая инфраструктура.

#  **Зачем нужен Vagrant**

###  1. Одинаковые окружения для всех

Забираем «работает у меня — не работает у тебя».  
Весь проект поднимается от одного файла.

###  2. Автоматизация создания VM

Одна команда:

```
vagrant up
```

и в нужной виртуалке поднимается:

- Linux/Windows
- нужная версия Python/Node/JDK
- сервисы (Nginx, PostgreSQL, Redis)
- настроенная сеть и порты
###  3. Не нужно держать окружение на хосте

Код — в хостовой системе, а всё окружение — внутри VM.
###  4. Быстрый откат окружения

Весь окружение хранится в конфигурации → всегда можно пересоздать.

#  **Из чего состоит Vagrant**

### 1. **Vagrantfile**

Файл конфигурации (Ruby-like синтаксис).  
Именно он описывает:

- базовый образ
- ресурсы VM
- сеть
- папки
- provisioning
### 2. **Box**

Это шаблон виртуальной машины:  
ОС + базовые настройки.

Примеры:

- ubuntu/jammy64
- debian/bookworm64
- centos/7

### 3. **Provider**

Средство виртуализации: VirtualBox, VMware, Docker, Libvirt.

### 4. **Provisioning**

Автоматическая настройка VM:

- shell-скрипты
- Ansible
- Puppet
- Chef
- Salt

#  **Простой пример Vagrantfile**

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.synced_folder "./app", "/var/www/app"

  config.vm.provision "shell", inline: <<-SHELL
    apt update
    apt install -y nginx
  SHELL
end
```

#  **Команды Vagrant**

|Команда|Что делает|
|---|---|
|`vagrant up`|Создать и запустить VM|
|`vagrant halt`|Остановить VM|
|`vagrant destroy`|Удалить VM|
|`vagrant ssh`|Зайти внутрь VM|
|`vagrant reload`|Перезагрузить, применяя изменения Vagrantfile|
|`vagrant provision`|Заново выполнить provisioning|
|`vagrant status`|Показать состояние|

#  **Сетевые режимы**

###  Port forwarding

Пробрасываем порты:

- host:8080 → guest:80

###  Private network

Сетевое взаимодействие хост ↔ VM.

```
config.vm.network "private_network", ip: "192.168.56.10"
```

###  Public network

VM получает IP из локальной сети.

#  **Синхронизация папок**

По умолчанию:

```
проекта → /vagrant
```

Можно подключать дополнительные:

```
config.vm.synced_folder "./src", "/home/vagrant/src"
```

#  **Преимущества Vagrant**

- стабильные, повторяемые окружения
- лёгкость использования
- автоматизация provisioning
- возможность работать с разными провайдерами
- удобно для разработки, CI и тестирования

#  **Недостатки Vagrant**

- более тяжёлый, чем Docker
- многие задачи проще делать в контейнерах
- VirtualBox на macOS/Windows может работать нестабильно
- медленнее Docker при поднятии окружений

#  **Когда лучше использовать Vagrant**

 Нужна полноценная VM (kernel, systemd)  
 Нужно эмулировать целый сервер, а не контейнер  
 Нужны разные Linux-дистрибутивы  
 DevOps обучение, тестирование Ansible/Puppet  
 Локальный mini-production стенд

#  **Когда Vagrant лучше заменить Docker**

Если:

- приложение работает в контейнерах
- нужна лёгкость и скорость
- требуется переносимость в CI/CD

→ Docker/docker-compose/Kubernetes будут лучше.
