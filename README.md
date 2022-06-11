# Роль Ansible: PostgreSQL

Устанавливает и настраивает сервер PostgreSQL на серверах RHEL/CentOS или Debian/Ubuntu.

## Требования

Нет особых требований; обратите внимание, что для этой роли требуется root-доступ, поэтому либо запустите ее в плейбуке с глобальным `become: yes`, либо вызовите роль в своем плейбуке, например:
    - hosts: database
      roles:
        - role: geerlingguy.postgresql
          become: yes

## Ролевые переменные

Доступные переменные перечислены ниже вместе со значениями по умолчанию (см. defaults/main.yml):

    postgresql_enablerepo: ""

(Только для RHEL/CentOS) Вы можете указать репозиторий, который будет использоваться для установки PostgreSQL, передав его здесь.
    postgresql_restarted_state: "restarted"

Задайте состояние службы при внесении изменений в конфигурацию. Рекомендуемые значения: «перезапущено» или «перезагружено».

    postgresql_python_library: python-psycopg2

Библиотека, используемая Ansible для связи с PostgreSQL. Если вы используете Python 3 (например, установленный через `ansible_python_interpreter`), вы должны изменить это на `python3-psycopg2`.

    postgresql_user: postgres
    postgresql_group: postgres

Пользователь и группа, под которыми будет работать PostgreSQL.

    postgresql_unix_socket_directories:
      - /var/run/postgresql

Каталоги (обычно один, но может быть и несколько), в которых будет создан сокет PostgreSQL.

    postgresql_service_state: started
    postgresql_service_enabled: true

Управляйте состоянием службы postgresql и тем, должна ли она запускаться во время загрузки.

    postgresql_global_config_options:
      - option: unix_socket_directories
        value: '{{ postgresql_unix_socket_directories | join(",") }}'

Глобальные параметры конфигурации, которые будут установлены в `postgresql.conf`. Обратите внимание, что для RHEL/CentOS 6 (или очень старых версий PostgreSQL) вам необходимо, по крайней мере, переопределить эту переменную и установить `option` в `unix_socket_directory`.

    postgresql_hba_entries:
      - { type: local, database: all, user: postgres, auth_method: peer }
      - { type: local, database: all, user: all, auth_method: peer }
      - { type: host, database: all, user: all, address: '127.0.0.1/32', auth_method: md5 }
      - { type: host, database: all, user: all, address: '::1/128', auth_method: md5 }

Настройте записи [аутентификация на основе хоста](https://www.postgresql.org/docs/current/static/auth-pg-hba-conf.html), которые будут установлены в `pg_hba.conf`. Варианты для записей включают в себя:

  - `type` (требуется)
  - `database` (требуется)
  - `user` (требуется)
  - `address` (требуется один из этих или следующих двух)
  - `ip_address`
  - `ip_mask`
  - `auth_method` (требуется)
  - `auth_options` (требуется)

При переопределении убедитесь, что вы скопировали все существующие записи из `defaults/main.yml`, если вам нужно сохранить существующие записи.

    postgresql_locales:
      - 'en_US.UTF-8'

(Только для Debian/Ubuntu) Используется для создания локалей, используемых базами данных PostgreSQL.

    postgresql_databases:
      - name: exampledb # required; the rest are optional
        lc_collate: # defaults to 'en_US.UTF-8'
        lc_ctype: # defaults to 'en_US.UTF-8'
        encoding: # defaults to 'UTF-8'
        template: # defaults to 'template0'
        login_host: # defaults to 'localhost'
        login_password: # defaults to not set
        login_user: # defaults to 'postgresql_user'
        login_unix_socket: # defaults to 1st of postgresql_unix_socket_directories
        port: # defaults to not set
        owner: # defaults to postgresql_user
        state: # defaults to 'present'

Список баз данных, которые необходимо убедиться, что они существуют на сервере. Требуется только `имя`; все остальные свойства необязательны.

    postgresql_users:
      - name: jdoe #required; the rest are optional
        password: # defaults to not set
        encrypted: # defaults to not set
        priv: # defaults to not set
        role_attr_flags: # defaults to not set
        db: # defaults to not set
        login_host: # defaults to 'localhost'
        login_password: # defaults to not set
        login_user: # defaults to '{{ postgresql_user }}'
        login_unix_socket: # defaults to 1st of postgresql_unix_socket_directories
        port: # defaults to not set
        state: # defaults to 'present'

Список пользователей, которые должны существовать на сервере. Требуется только `name`; все остальные свойства необязательны.

    postgres_users_no_log: true

Следует ли выводить пользовательские данные (которые могут содержать конфиденциальную информацию, например пароли) при управлении пользователями.

    postgresql_version: [OS-specific]
    postgresql_data_dir: [OS-specific]
    postgresql_bin_path: [OS-specific]
    postgresql_config_path: [OS-specific]
    postgresql_daemon: [OS-specific]
    postgresql_packages: [OS-specific]

248 / 5 000
Результаты перевода
star_border
Переменные, зависящие от ОС, которые устанавливаются включаемыми файлами в каталоге `vars` этой роли. Их не следует переопределять, если только вы не используете версию PostgreSQL, которая не была установлена с помощью системных пакетов.

## Зависимости

Нет.

## Пример Playbook

    - hosts: database
      become: yes
      vars_files:
        - vars/main.yml
      roles:
        - geerlingguy.postgresql

*Включение `vars/main.yml`*:

    postgresql_databases:
      - name: example_db
    postgresql_users:
      - name: example_user
        password: supersecure
