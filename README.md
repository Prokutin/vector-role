# vector-role

Роль Ansible для установки [Vector](https://vector.dev) из официального архива, настройки конфигурации и запуска сервиса `vector` (systemd). Логи из файлов на хосте отправляются в таблицу ClickHouse.

## Переменные

### defaults/main.yml 

| Переменная | По умолчанию | Описание |
|---|---|---|
| `vector_version` | `"0.34.1"` | Версия Vector |
| `vector_install_dir` | `"/opt"` | Каталог, куда распаковывается архив |
| `vector_download_dir` | `"/tmp"` | Каталог для скачивания архива на хосте |
| `vector_config_dir` | `"/etc/vector"` | Каталог конфигурации (`vector.yaml`) |
| `vector_data_dir` | `"/var/lib/vector"` | Каталог данных Vector (`data_dir`) |
| `vector_log_paths` | `["/var/log/*.log"]` | Файлы-источники логов |
| `vector_clickhouse_endpoint` | `"http://{{ hostvars['clickhouse'].internal_ip }}:8123"` | HTTP-адрес ClickHouse для sink |
| `vector_clickhouse_database` | `"logs"` | База данных в ClickHouse |
| `vector_clickhouse_table` | `"vector_logs"` | Таблица в ClickHouse |

### vars/main.yml 

| Переменная | Значение | Описание |
|---|---|---|
| `vector_archive_dir` | `vector-x86_64-unknown-linux-gnu` | Каталог внутри архива |
| `vector_download_url` | собирается из `vector_version` | URL архива |
| `vector_archive` | `{{ vector_download_dir }}/vector-{{ vector_version }}.tar.gz` | Путь к скачанному архиву |
| `vector_bin_source` | `{{ vector_install_dir }}/{{ vector_archive_dir }}/bin/vector` | Исполняемый файл после распаковки |
| `vector_bin_path` | `/usr/local/bin/vector` | Символическая ссылка в PATH |
| `vector_service_name` | `vector` | Имя systemd-сервиса |

## Шаблоны

- `templates/vector.yaml.j2` — конфиг Vector: источник `file` и sink `clickhouse`.
- `templates/vector.service.j2` — unit-файл systemd.

## Теги

- `vector_install` — скачивание, распаковка и ссылка на бинарник;
- `vector_config` — каталоги, конфиг и unit-файл;
- `vector_service` — запуск и автозапуск сервиса.

## Пример использования

```yaml
- hosts: vector
  become: true
  roles:
    - role: vector-role
      vars:
        vector_log_paths:
          - "/var/log/*.log"
          - "/var/log/nginx/*.log"
```

Проверка без изменений на хостах:

```bash
ansible-playbook site.yml --check --diff
```

В режиме `--check` скачивание архива выполняется, а распаковка, ссылка, запуск сервиса и обработчики пропускаются.

## Зависимости

Нет. Роль не зависит от других ролей, но ожидает, что ClickHouse уже установлен и принимает подключения на порту 8123 (см. роль `clickhouse`).

