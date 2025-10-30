
##  Что такое Ansible

**Ansible** — это **инструмент автоматизации**, созданный для:

- управления конфигурациями серверов;
- оркестрации (управления множеством систем одновременно);
- деплоя приложений;
- выполнения одноразовых задач (например, обновления пакетов).

Он написан на **Python** и использует **SSH** (или WinRM для Windows), чтобы подключаться к удалённым машинам — **без установки агентов** (в отличие от [[Puppet]], [[Chef]], [[SaltStack]]).


##  Основные особенности

| Особенность          | Описание                                                                                   |
| -------------------- | ------------------------------------------------------------------------------------------ |
| Agentless            | Не требует агента на управляемых узлах — достаточно SSH.                                   |
| Идемпотентность      | Повторный запуск не изменит систему, если она уже в нужном состоянии.                      |
| Язык YAML            | Конфигурации описываются декларативно в виде _playbooks_ на YAML.                          |
| Расширяемость        | Можно писать собственные модули на Python или использовать готовые.                        |
| Кроссплатформенность | Управляет Linux, macOS, Windows, сетевыми устройствами, облаками (AWS, GCP, Azure и т.д.). |

##  Архитектура Ansible

**1. Control Node (управляющий узел)**  
Машина, на которой установлен Ansible. Отсюда запускаются команды и playbooks.

**2. Managed Nodes (управляемые узлы)**  
Серверы, к которым Ansible подключается по SSH/WinRM.

**3. Inventory (инвентарь)**  
Файл со списком хостов и групп, которыми управляет Ansible.  

```ini
[web]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[db]
db1 ansible_host=192.168.1.12
```

**4. Playbooks**  
YAML-файлы, описывающие, какие задачи выполнить.  

```yaml
- name: Установка Nginx
  hosts: web
  become: yes
  tasks:
    - name: Установить пакет nginx
      apt:
        name: nginx
        state: present
```

**5. Modules (модули)**  
Готовые действия: `apt`, `copy`, `template`, `user`, `service`, `shell`, и т.д.  
Ты просто указываешь, какой модуль использовать и с какими параметрами.

**6. Roles (роли)**  
Способ структурировать большие проекты.  
Роль содержит свои `tasks`, `templates`, `handlers`, `vars`, `files`, и т.д.  
Структура:

```
roles/
  nginx/
    tasks/main.yml
    templates/nginx.conf.j2
    handlers/main.yml
    vars/main.yml
```

**7. Handlers**  
Это “реакции” на события, например, перезапуск сервиса после изменения конфигурации.

```yaml
tasks:
  - name: Обновить конфиг nginx
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify:
      - Перезапустить nginx

handlers:
  - name: Перезапустить nginx
    service:
      name: nginx
      state: restarted
```

##  Ansible Vault

Используется для шифрования чувствительных данных:

```bash
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
ansible-vault view secrets.yml
ansible-playbook playbook.yml --ask-vault-pass
```

---

##  Ansible и облака

Ansible имеет коллекции модулей для:

- **AWS** (`amazon.aws.ec2`, `amazon.aws.s3`)
- **Azure**, **GCP**, **OpenStack**
- Kubernetes (`community.kubernetes`)
- Docker (`community.docker`)

Пример для AWS:

```yaml
- name: Создать EC2 инстанс
  hosts: localhost
  tasks:
    - name: Запуск экземпляра
      amazon.aws.ec2_instance:
        key_name: mykey
        instance_type: t2.micro
        image_id: ami-0c94855ba95c71c99
        wait: yes
```

##  Интеграции

Ansible часто используется вместе с:

- **[[Jenkins]] / [[GitLab]] CI** → автоматизация развёртываний;
- **Terraform** → управление инфраструктурой ([[Terraform]] создаёт ресурсы, Ansible их настраивает);
- **[[Prometheus]] / [[Grafana]]** → настройка мониторинга;
- **[[Docker]] / [[Kubernetes]]** → управление контейнерами и кластером.

## Пример реального сценария

```yaml
- name: Настройка веб-сервера
  hosts: web
  become: yes

  vars:
    nginx_port: 8080

  tasks:
    - name: Установить Nginx
      apt:
        name: nginx
        state: present

    - name: Скопировать конфиг
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Перезапустить nginx

  handlers:
    - name: Перезапустить nginx
      service:
        name: nginx
        state: restarted
```

##  1. Проверка и отладка соединения

|Команда|Описание|
|---|---|
|`ansible all -m ping`|Проверить доступность всех хостов из inventory.|
|`ansible web -m ping`|Проверить доступность группы `web`.|
|`ansible db -m shell -a "uptime"`|Выполнить команду `uptime` на всех хостах из группы `db`.|
|`ansible all -m setup`|Собрать и показать “факты” (facts) о хостах — ОС, IP, память и т.д.|
|`ansible all -m gather_facts`|То же самое, что `setup`, но используется внутри playbooks.|
|`ansible -m ping -i inventory.ini all -u ubuntu --private-key ~/.ssh/id_rsa`|Проверить соединение с указанием пользователя и ключа.|

##  2. Работа с Playbooks

|Команда|Описание|
|---|---|
|`ansible-playbook playbook.yml`|Запуск playbook.|
|`ansible-playbook -i inventory.ini playbook.yml`|Запуск с указанием inventory.|
|`ansible-playbook playbook.yml --check`|**Dry-run**: показать, что изменится, без применения.|
|`ansible-playbook playbook.yml --diff`|Показать различия (например, изменённые файлы).|
|`ansible-playbook playbook.yml -v`|Verbose-режим (больше вывода).|
|`ansible-playbook playbook.yml -vvv`|Подробный отладочный вывод (максимум информации).|
|`ansible-playbook playbook.yml --tags "install,config"`|Запустить только задачи с указанными тегами.|
|`ansible-playbook playbook.yml --skip-tags "cleanup"`|Пропустить задачи с определёнными тегами.|
|`ansible-playbook playbook.yml --limit web`|Запустить playbook только на группе или конкретном хосте.|
|`ansible-playbook playbook.yml --start-at-task "Установить Nginx"`|Начать выполнение с определённой задачи.|
|`ansible-playbook playbook.yml --list-hosts`|Показать, на каких хостах будет выполняться playbook.|
|`ansible-playbook playbook.yml --list-tasks`|Показать список всех задач в playbook.|
|`ansible-playbook playbook.yml --step`|Запрашивать подтверждение перед каждой задачей.|

##  3. Работа с Inventory

|Команда|Описание|
|---|---|
|`ansible-inventory --list`|Показать всё содержимое inventory в JSON-формате.|
|`ansible-inventory --graph`|Отобразить иерархию групп и хостов.|
|`ansible-inventory -i inventory.ini --host web1`|Показать данные конкретного хоста.|
|`ansible all --list-hosts`|Список всех хостов, к которым будет применяться команда.|
|`ansible-inventory -i dynamic_inventory.py --list`|Проверить динамический inventory.|

##  4. Работа с модулями

|Команда|Описание|
|---|---|
|`ansible-doc -l`|Список всех доступных модулей.|
|`ansible-doc -s apt`|Показать пример использования модуля `apt`.|
|`ansible-doc copy`|Подробное описание модуля `copy`.|
|`ansible all -m command -a "ls /tmp"`|Выполнить команду без shell.|
|`ansible all -m shell -a "echo Hello > /tmp/test.txt"`|Выполнить команду в shell.|
|`ansible all -m file -a "path=/tmp/test.txt state=absent"`|Удалить файл.|
|`ansible localhost -m debug -a "msg='Hello Ansible'"`|Вывести сообщение отладки.|

##  5. Работа с Vault (шифрование секретов)

|Команда|Описание|
|---|---|
|`ansible-vault create secrets.yml`|Создать зашифрованный файл.|
|`ansible-vault edit secrets.yml`|Отредактировать зашифрованный файл.|
|`ansible-vault view secrets.yml`|Просмотреть содержимое.|
|`ansible-vault encrypt file.yml`|Зашифровать существующий файл.|
|`ansible-vault decrypt file.yml`|Расшифровать файл.|
|`ansible-vault rekey secrets.yml`|Изменить пароль шифрования.|
|`ansible-playbook playbook.yml --ask-vault-pass`|Ввести пароль при запуске playbook.|
|`ansible-playbook playbook.yml --vault-password-file .vault_pass`|Указать файл с паролем.|

##  6. Работа с ролями и коллекциями

|Команда|Описание|
|---|---|
|`ansible-galaxy init nginx`|Создать новую роль с шаблоном структуры.|
|`ansible-galaxy list`|Список установленных ролей.|
|`ansible-galaxy install geerlingguy.nginx`|Установить роль с Galaxy.|
|`ansible-galaxy collection install community.docker`|Установить коллекцию модулей.|
|`ansible-galaxy collection list`|Список установленных коллекций.|
|`ansible-galaxy collection build`|Собрать свою коллекцию.|
|`ansible-galaxy collection publish my_collection.tar.gz`|Опубликовать коллекцию.|

##  7. Отладка и тестирование

|Команда|Описание|
|---|---|
|`ansible-playbook playbook.yml --syntax-check`|Проверить синтаксис YAML.|
|`ansible-playbook playbook.yml --check --diff`|Проверить, что изменится, без выполнения.|
|`ansible-playbook playbook.yml -v`|Подробный вывод.|
|`ansible-playbook playbook.yml --list-tags`|Показать все теги в playbook.|
|`ansible localhost -m debug -a "var=hostvars['web1']"`|Посмотреть переменные конкретного хоста.|
|`ansible localhost -m include_vars -a "dir=group_vars"`|Загрузить все переменные из каталога.|

## 8. Полезные комбинации

```bash
# Проверить соединение с хостами через указанный inventory
ansible -i inventory.ini all -m ping

# Выполнить команду с повышением прав
ansible web -b -m shell -a "systemctl restart nginx"

# Применить playbook к одному хосту
ansible-playbook -i inventory.ini playbook.yml --limit web1

# Проверить “факты” о конкретном хосте
ansible web1 -m setup | less
```

##  Полезные ресурсы

- 🔗 [Официальная документация](https://docs.ansible.com/)
- 📦 [Ansible Galaxy](https://galaxy.ansible.com/) — хранилище ролей
- 📚 Книга: _Ansible for DevOps_ (Jeff Geerling)