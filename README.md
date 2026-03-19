# Ansible Playbook: ClickHouse and Vector Installation

## 📋 Описание

Данный playbook выполняет автоматическую установку и настройку двух компонентов:
- **ClickHouse**
- **Vector**

Playbook предназначен для Red Hat-совместимых дистрибутивов (CentOS/RHEL/AlmaLinux/Rocky).

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

5️⃣ Запуск ansible-lint

$ ansible-lint site.yml

6️⃣ Запуск с флагом --check

$ ansible-playbook -i inventory/prod.yml site.yml --check

7️⃣ Первый запуск с --diff

$ ansible-playbook -i inventory/prod.yml site.yml --diff

8️⃣ Второй запуск с --diff (проверка идемпотентности)

$ ansible-playbook -i inventory/prod.yml site.yml --diff

📸 Скриншоты выполнения
*Скриншот 1: Запуск ansible-lint*
https://screenshots/ansible-lint.png

*Скриншот 2: Запуск с --check*
https://screenshots/check-mode.png

*Скриншот 3: Первый запуск с --diff*
https://screenshots/first-diff.png

*Скриншот 4: Второй запуск с --diff (идемпотентность)*
https://screenshots/second-diff.png

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
