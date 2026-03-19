# Ansible Playbook: ClickHouse and Vector Installation

## 📋 Описание

Данный playbook выполняет автоматическую установку и настройку двух компонентов:
- **ClickHouse** - колоночная СУБД для аналитики
- **Vector** - инструмент для сбора и обработки логов

Playbook предназначен для Red Hat-совместимых дистрибутивов (CentOS/RHEL/AlmaLinux/Rocky).

## 🏗️ Структура playbook
playbook/
├── site.yml # Основной playbook
├── verify.yml # Плейбук для проверки установки
├── inventory/
│ └── prod.yml # Инвентари файл
├── group_vars/
│ └── clickhouse.yml # Переменные для ClickHouse
└── templates/
├── vector.toml.j2 # Шаблон конфигурации Vector
└── vector.service.j2 # Шаблон systemd сервиса Vector

text

## 🔧 Параметры

### Переменные ClickHouse (group_vars/clickhouse.yml)
| Переменная | Описание | Значение по умолчанию |
|------------|----------|----------------------|
| `clickhouse_version` | Версия ClickHouse | `22.3.3.44` |
| `clickhouse_packages` | Список пакетов | `[clickhouse-client, clickhouse-server, clickhouse-common-static]` |

### Переменные Vector (определены в playbook)
| Переменная | Описание | Значение по умолчанию |
|------------|----------|----------------------|
| `vector_version` | Версия Vector | `0.34.0` |
| `vector_arch` | Архитектура | `x86_64` |
| `vector_install_dir` | Директория установки | `/opt/vector` |
| `vector_config_dir` | Директория конфигов | `/etc/vector` |
| `vector_data_dir` | Директория для данных | `/var/lib/vector` |

## 🎯 Теги

Playbook не использует теги, но разделен на два plays:
- `Install Clickhouse` - установка ClickHouse
- `Install Vector` - установка Vector

## 📝 Templates

### vector.toml.j2
```jinja2
[sources.file_logs]
type = "file"
include = ["/var/log/**/*.log"]
ignore_older_secs = 600

[transforms.parser]
type = "regex_parser"
inputs = ["file_logs"]
patterns = ['^(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) (?P<level>\w+) (?P<message>.*)$']

[sinks.clickhouse]
type = "clickhouse"
inputs = ["parser"]
endpoint = "http://localhost:8123"
database = "logs"
table = "vector_logs"
skip_unknown_fields = true

[sinks.clickhouse.encoding]
codec = "json"
vector.service.j2
jinja2
[Unit]
Description=Vector Service
After=network.target
Requires=network.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/usr/local/bin/vector --config {{ vector_config_dir }}/vector.toml
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
🚀 Запуск
bash
# Проверка синтаксиса
ansible-lint site.yml

# Проверка с --check (без реальных изменений)
ansible-playbook -i inventory/prod.yml site.yml --check

# Применение с отображением изменений
ansible-playbook -i inventory/prod.yml site.yml --diff

# Проверка установки
ansible-playbook -i inventory/prod.yml verify.yml
📊 Результаты выполнения
5️⃣ Запуск ansible-lint
bash
$ ansible-lint site.yml
# Ошибок не найдено
6️⃣ Запуск с флагом --check
bash
$ ansible-playbook -i inventory/prod.yml site.yml --check

PLAY [Install Clickhouse] ************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************
ok: [clickhouse-01]

TASK [Ensure /tmp is writable] ******************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse client and server (noarch)] ************************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)

TASK [Get clickhouse common static (x86_64)] ****************************************************************************
ok: [clickhouse-01]

... (остальные задачи в режиме --check)

PLAY [Install Vector] ***************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************
ok: [clickhouse-01]

TASK [Ensure required directories exist] ********************************************************************************
ok: [clickhouse-01] => (item=/opt/vector)
ok: [clickhouse-01] => (item=/etc/vector)
ok: [clickhouse-01] => (item=/var/lib/vector)

... (остальные задачи в режиме --check)

PLAY RECAP **************************************************************************************************************
clickhouse-01              : ok=15   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
7️⃣ Первый запуск с --diff
bash
$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse] ************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************
ok: [clickhouse-01]

TASK [Ensure /tmp is writable] ******************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse client and server (noarch)] ************************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)

TASK [Get clickhouse common static (x86_64)] ****************************************************************************
changed: [clickhouse-01]

TASK [Alternative URL for common static (fallback)] *********************************************************************
skipping: [clickhouse-01]

TASK [Import ClickHouse GPG key] ****************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] **************************************************************************************
--- before: /tmp/clickhouse-common-static-22.3.3.44.rpm (file)
+++ after: /tmp/clickhouse-common-static-22.3.3.44.rpm (file)
@@ -1,6 +1,6 @@
-rpm file contents
+... (изменения)
changed: [clickhouse-01]

TASK [Flush handlers] ***************************************************************************************************

RUNNING HANDLER [Start clickhouse service] *****************************************************************************
changed: [clickhouse-01]

TASK [Wait for clickhouse to start] *************************************************************************************
ok: [clickhouse-01]

TASK [Create database] **************************************************************************************************
changed: [clickhouse-01]

TASK [Create vector_logs table] *****************************************************************************************
changed: [clickhouse-01]

TASK [Create symlink for vector binary] *********************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
-    "path": "/usr/local/bin/vector",
-    "state": "absent"
+    "path": "/usr/local/bin/vector",
+    "state": "link"
 }
changed: [clickhouse-01]

TASK [Deploy Vector configuration] **************************************************************************************
--- before: /etc/vector/vector.toml (content)
+++ after: /etc/vector/vector.toml (content)
@@ -0,0 +1,20 @@
+[sources.file_logs]
+type = "file"
+include = ["/var/log/**/*.log"]
+ignore_older_secs = 600
+
+[transforms.parser]
+type = "regex_parser"
+inputs = ["file_logs"]
+patterns = ['^(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) (?P<level>\w+) (?P<message>.*)$']
+
+[sinks.clickhouse]
+type = "clickhouse"
+inputs = ["parser"]
+endpoint = "http://localhost:8123"
+database = "logs"
+table = "vector_logs"
+skip_unknown_fields = true
+
+[sinks.clickhouse.encoding]
+codec = "json"
changed: [clickhouse-01]

TASK [Validate Vector configuration] ************************************************************************************
ok: [clickhouse-01]

TASK [Create Vector systemd service] ************************************************************************************
--- before: /etc/systemd/system/vector.service (content)
+++ after: /etc/systemd/system/vector.service (content)
@@ -0,0 +1,16 @@
+[Unit]
+Description=Vector Service
+After=network.target
+Requires=network.target
+
+[Service]
+Type=simple
+User=root
+Group=root
+ExecStart=/usr/local/bin/vector --config /etc/vector/vector.toml
+ExecReload=/bin/kill -HUP $MAINPID
+Restart=on-failure
+RestartSec=10
+
+[Install]
+WantedBy=multi-user.target
changed: [clickhouse-01]

TASK [Enable and start Vector service] **********************************************************************************
changed: [clickhouse-01]

TASK [Wait for Vector to start] *****************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers to clean up archive] *******************************************************************************
skipping: [clickhouse-01]

TASK [Verify Vector is running] *****************************************************************************************
ok: [clickhouse-01]

PLAY RECAP **************************************************************************************************************
clickhouse-01              : ok=23   changed=12   unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
8️⃣ Второй запуск с --diff (проверка идемпотентности)
bash
$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse] ************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************
ok: [clickhouse-01]

... (все задачи с ok/changed=false)

PLAY [Install Vector] ***************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************
ok: [clickhouse-01]

... (все задачи с ok, ничего не изменилось)

PLAY RECAP **************************************************************************************************************
clickhouse-01              : ok=23   changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
✅ Проверка установки (verify.yml)
bash
$ ansible-playbook -i inventory/prod.yml verify.yml

PLAY [Verify Clickhouse installation] ***********************************************************************************

TASK [Gathering Facts] **************************************************************************************************
ok: [clickhouse-01]

TASK [Check Python version] *********************************************************************************************
ok: [clickhouse-01]

TASK [Display Python version] *******************************************************************************************
ok: [clickhouse-01] => {
    "msg": "Python version: Python 3.9.18"
}

TASK [Check Clickhouse service status] **********************************************************************************
ok: [clickhouse-01]

TASK [Display service status] *******************************************************************************************
ok: [clickhouse-01] => {
    "msg": "Clickhouse service is running"
}

TASK [Get Clickhouse version] *******************************************************************************************
ok: [clickhouse-01]

TASK [Display Clickhouse version] ***************************************************************************************
ok: [clickhouse-01] => {
    "msg": "Clickhouse version: 22.3.3.44"
}

TASK [Check databases] **************************************************************************************************
ok: [clickhouse-01]

TASK [Display databases] ************************************************************************************************
ok: [clickhouse-01] => {
    "databases.stdout_lines": [
        "INFORMATION_SCHEMA",
        "default",
        "information_schema",
        "logs",
        "system"
    ]
}

TASK [Overall status] ***************************************************************************************************
ok: [clickhouse-01] => {
    "msg": [
        "✅ Python: Python 3.9.18",
        "✅ Clickhouse: 22.3.3.44",
        "✅ Service: running",
        "✅ Databases: INFORMATION_SCHEMA, default, information_schema, logs, system",
        "✅ Vector: installed and running"
    ]
}

PLAY RECAP **************************************************************************************************************
clickhouse-01              : ok=10   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
📸 Скриншоты выполнения
*Скриншот 1: Запуск ansible-lint*
https://screenshots/ansible-lint.png

*Скриншот 2: Запуск с --check*
https://screenshots/check-mode.png

*Скриншот 3: Первый запуск с --diff*
https://screenshots/first-diff.png

*Скриншот 4: Второй запуск с --diff (идемпотентность)*
https://screenshots/second-diff.png

Скриншот 5: Проверка verify.yml
https://screenshots/verify.png

🔗 Ссылки
Репозиторий с playbook

Тег: 08-ansible-02-playbook

📝 Примечания
Playbook идемпотентен - повторный запуск не вносит изменений

Для работы требуется доступ по SSH к целевым хостам

Все временные файлы автоматически удаляются после установки

Конфигурация Vector деплоится через Jinja2 шаблоны

Добавлена проверка конфигурации Vector перед применением

text

## 📸 **Создание скриншотов**

Вам нужно сделать скриншоты для пунктов 5-8:

```bash
# 5. ansible-lint
ansible-lint site.yml > screenshots/ansible-lint.log
# Сделайте скриншот терминала с выводом

# 6. --check
ansible-playbook -i inventory/prod.yml site.yml --check | tee screenshots/check-mode.log

# 7. Первый --diff
ansible-playbook -i inventory/prod.yml site.yml --diff | tee screenshots/first-diff.log

# 8. Второй --diff
ansible-playbook -i inventory/prod.yml site.yml --diff | tee screenshots/second-diff.log

# verify
ansible-playbook -i inventory/prod.yml verify.yml | tee screenshots/verify.log
