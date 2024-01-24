Vector-role
=========

Роль выполняет:
1. Установку Vector из архива с дистрибутивом 
2. Первичную конфигурацию Vector с демо конфигурацией по генерации событий syslog и отправку данные в БД Clickhouse


Requirements
------------

1. Centos ОС.
2. Предустановленная БД Clickhouse.

Role Variables
--------------

`defaults/main.yml`:

vector_version - Версия используемого ПО Vector. По умолчанию 0.35.0-1

```
vector_config:
  sources:
    logs_logs:
      type: demo_logs
      format: syslog
      interval: 1
  transforms:
    parse_logs:
      inputs:
        - logs_logs
      source: |-
        . = parse_syslog!(string!(.message))
        .timestamp = to_string(.timestamp)
        .timestamp = slice!(.timestamp, start:0, end: -1)
      type: remap
  sinks:
    to_clickhouse:
      type: clickhouse
      inputs:
        - parse_logs
      database: vector_logs
      endpoint: "http://ip_address:8123"
      table: logs_logs
      compression: gzip
  api:
    enabled: true
    address: "0.0.0.0:8686"
```

Dependencies
------------

Для установки Clickhouse требуется роль ansible-clickhouse: https://github.com/AlexeySetevoi/ansible-clickhouse

Example Playbook
----------------

Пример добавления роли в playbook:

```
- name: Install Vector
  tags: [vector]
  hosts: vector
  roles:
    - vector-role
```

License
-------

BSD
