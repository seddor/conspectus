# Ansible

Ansible — ситема управления конфигурациями, позволяет автоматизировать настройку и развёртывания ПО.

## Ad-Hoc

У ansible есть несколько режимов работы, самый простой из них — ad-hoc, когда запрос к серверу выполняется напрямую из командной строки, без создания дополнительных файлов.

```bash
# Проверяет доступность сервера по ip-адресу
# all – запрос выполняется для всех указанных машин
# 157.230.82.133 – ip-адрес моей машины. В вашем случае будет другой
# запятая в конце не ошибка, так нужно для работы Ansible
# root – пользователь для подключения по ssh
# ping – используемая команда (модуль ansible)
ansible all -i '157.230.82.133, ' -u root -m ping
```

В примере выше происходит подключение по ssh к удалённому серверу и выполнение команды `ping`, в ansible за запуск этой команды отвечает отдельный модуль `ping`, модули в Ansbile выполняют команды на удалённых машинах. Один из модулей — `command` позволяет запускать произвольную команду на сервере.

```bash
# Модуль command запускает на удаленной машине команду uptime
# -a – означает аргумент. В случае модуля command это то, что запустится на удаленной машине
ansible all -i '157.230.82.133, ' -u root -m command -a 'uptime'
157.230.82.133 | CHANGED | rc=0 >>
 06:12:48 up 11 days, 21:53,  1 user,  load average: 0.04, 0.04, 0.01
```

## Файл инвентаризации

В ansible есть специальный файл инвентаризации, который описывает инфраструктуру, в этом файле можно указывать адреса серверов:

```ini
157.230.82.133
# Именем может быть имя домена, что удобнее
# имена понятнее и не завязаны на ip-адрес, который может поменяться
ec2-3-17-186-111.us-east-2.compute.amazonaws.com
# тут другой адрес
# тут еще один
```

Файл имеет расширение `.ini`, однако у него нет строго заданного имени. Запуск ansible с файлом инвентаризации:

```bash
# -i – означает inventory, all — для всех серверов
ansible all -i inventory.ini -u root -m ping
```

Ansible выполняет команды к серверам параллельно, при этом ошибка выполнения команды на одном сервере не ведёт к остановке команды, после выполнения ansible покажет на каких серверах команды запустилась удачно, а на каких нет.

В файле инвентаризации сервера можно выделять в группы серверов:

```ini
[webservers]
ec2-3-17-186-111.us-east-2.compute.amazonaws.com
ec2-3-20-186-111.us-east-2.compute.amazonaws.com

[appservers]
157.230.82.133
# кастомный порт, по дефолту ssh находиться на 22 порту, в иных случаях нужно указывать порт явно
ec2-52-17-124-244.eu-west:8081
```

Для выполнения команды на группе нужно указать её назнавние при запуске ansible:

```bash
ansible appservers -i inventory.ini -a 'uptime'
```

Для серверов можно указывать кастомное имя с помощью ansible_host

```ini
jumper ansible_host=123.25.26.35
```

Затем это имя можно использовать в командах ansible с опцией `--limit`:

```bash
ansible all --limit jumper -i inventory.ini -m ping
```

Можно задать группировки для групп:

```ini
[servers:children]
webservers
appservers

[webservers]
ec2-52-17-124-244.eu-west:8081

[appservers]
ec2-52-17-124-245.eu-west
```

Это позволит делать запросы сразу к обоим группам:

```bash
ansible servers -i inventory.ini -m ping
```

Ansible может работать и локально, для этого в файле инвентаризации указывается:

```ini
localhost ansible_connection=local
```

Затем:

```bash
ansible all --limit localhost -i inventory.ini -u root -m ping
```

## Плейбук

Плейбук — это yaml файл, в котором указывается, какие задачи и на каких серверах будут выполняться:

```yaml
# На какой группе серверов
- hosts: webservers

  tasks:
    - name: install redis server
      # apt-get update && apt-get install redis-server
      ansible.builtin.apt: # имя модуля Ansible
        name: redis-server
        state: present
        update_cache: yes
      # запуск команды от root, если указать параметр become_user: то запуск будет от указанного пользователя
			become: yes
			
    - name: remove redis server
      # apt-get remove redis-server
      ansible.builtin.apt:
        name: redis-server
        state: absent
      # запуск команды от root
			become: yes   
```

В примере выше для группы серверов `webservers` выполняются две такси, установка и удаление редис-сервера.

Структура такси:

* Имя задачи — необязательный параметр, в котором описывается задача. Нужна только для вывода во время выполнения плейбука.
* Модуль и его параметры — определяют команду, которая будет выполнена на указанной группе серверов. В примере выше это `apt`.

Имя модуля начинается с префикса *ansible.builtin*. Ansible позволяет создавать свои модули.

Модули принимают параметры, для `apt` это:

* `name: redis-server` — имя пакета.
* `state: present` — состояние, в которое требуется привести модуль. Ansible убедится, что этот пакет есть, либо доустановит его.
* `update_cache: yes` — выполняет команду `apt-get update` для того, чтобы обновить информацию о пакетах в индексе. Во второй  задаче обновлять индекс не требуется. Достаточно указать состояние `state: absent`, чтобы Ansible просто удалил redis-сервер.

В плейбуке может быть сколько угодно таксов, но каждая таска выполняет только один модуль.

Для запуска плейбука используется команда `ansible-playbook`:

```bash
# В отличие от ad-hoc режима, группа хостов указывается внутри плейбука
ansible-playbook playbook.yml -i inventory.ini
```

## Теги

В плейбуках можно использовать теги для группировки каких-либо задач, для их использования указывается опция `tags`:

```yaml
- hosts: webservers
  tasks:
    - name: install nginx
      ansible.builtin.apt:
        name: nginx
        state: latest
      become: yes
      tags: nginx

    - name: install redis server
      ansible.builtin.apt:
        name: redis-server
        state: present
        update_cache: yes
      become: yes

    - name: update cron
      ansible.builtin.cron:
        name: "check dirs"
        minute: "0"
        hour: "5,2"
        job: "ls -alh > /dev/null"

    - name: update nginx config
      ansible.builtin.copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
      become: yes
      tags: [nginx, config]

    - name: restart nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
      become: yes
      tags: nginx
```

Для запуска ansible с определёнными тегами используется параметр `-t` (`--tags`):

```bash
ansible-playbook --check playbook.yml -i inventory.ini -t nginx
```

`--check` — указывает, что нужно выполнить проверки возможных изменений, без самого запуска команд.

Также можно использовать опцию `--skip-tags` для запуска задач, не помечных указанными тегами:

```bash
ansible-playbook --check playbook.yml -i inventory.ini --skip-tags nginx
```

## handlers (обработчики)

Обработчики предназначены для запуска каких-либо задач, при изменениях вызываных другой задачей, например необходимо перезапустить какой-либо сервис, после внесения изменений.

Обработчики определяются в корневой ноде `handlers`:

```yaml
- hosts: webservers
  tasks:
    - name: install nginx
      ansible.builtin.apt:
        name: nginx
        state: latest
      become: yes

    - name: update nginx config
      ansible.builtin.copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
      notify:
        - restart nginx
      become: yes

  handlers:
    - name: restart nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
      become: yes
```

Теперь, если задачи "update nginx config" произведёт какие-либо изменения, то это вызовет запуск обработчика "restart nginx". Важно помнить, что перезапуск будет произведён в конце, после выполнения остальных тасков плейбука, при этом он будет выполнен один раз, даже если его стригерят несколько тасков.

## Переменные

В плейбуках можно описывать переменные, которые потом можно будет использовать в тасках:

```yaml
- hosts: webservers
  vars:
    root_dir: /var/tmp/www
  tasks:
    - name: update nginix config
      ansible.builtin.template:
        src: templates/nginx.conf.j2
        dest: "{{root_dir}}/nginx.conf"

    - name: update index.html
      ansible.builtin.copy:
        src: files/index.html
        dest: "{{root_dir}}/index.html"
```

Ansible использует питоновский шаблонизатор **Jinja 2**. Который заменяет все `{{ название переменной }}` на значения переменных перед запуском yaml файла.

Модуль [ `ansible.builtin.template` ](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)  позволяет использовать перемменные ansible в при работе с кастомными файлами, таким обрзаом, если в файле указать:

```
server {
    location / {
        root {{root_dir}}
    }
}
```

То при запуске модуля `template`, `{{root_dir}}` будет заменено на значение переменной.

Переменные также можно использовать в файлах инвентаризации. Для того, чтобы установить переменную для конкретного хоста дотаточно после его имени написать название и значение переменной:

```ini
[webservers]
web1.example.com server_name=hexlet.io
web2.example.com

[appservers]
app1.example.com

# переменная применяется к группе хостов
[webservers:vars]
root_dir=/var/tmp
# переменная применяется ко всем хостам
[all:vars]
root_dir=/var/tmp
```

Полный список всех серверов, групп и переменных можно посмотреть с помощью команды `ansible-inventory` с флагами `--list` или `--graph`:

```bash
ansible-inventory -i inventory.ini --list
ansible-inventory -i inventory.ini --graph
```

## Факты

В Ansible есть понятие факты (facts) — это информация о тех машинах, на которых производиться запуск. Это статическая информация о сервере (ОС, железо, настройки сеи, переменные окружения и т.п.).

Все факты можно использовать в плейбуках, они хранятся в переменной `ansible_facts`:

```yaml
# Задаем переменную и сообщаем, что буфер должен использовать десятую часть от общего объема памяти
buffer: "{{ ansible_facts['memtotal_mb'] / 10 }}"
# По умолчанию Ansible регистрирует как переменные факты верхнего уровня
# поэтому для примера вше можно также использовать:
buffer: "{{ ansible_memtotal_mb / 10 }}"
```

При запуске плейбуков факты собираются при запуске первой задачи, однако, иногда это может мешать, т.к. сбор фактов не моментальная операция, если факты не нужны их сбор можно отключить:

```yaml
- hosts: webservers
  gather_facts: no
  tasks:
    - name: install redis server
      ansible.builtin.apt:
        name: redis-server
        state: present
        update_cache: yes
```

## Циклы

В плейбаках можно использовать циклы:

```yaml
- hosts: all
  tasks:
    - name: install packages
      ansible.builtin.apt:
        name: "{{ item }}"
        state: latest
      loop:
        - redis-server
        - postgresql
      become: yes
    # для apt использования цикла не обязательно, можно использовать массив
    - ansible.builtin.apt:
      state: latest
      name:
        - redis-server
        - postgresql

```

Здесь используется переменная `{{ item }}`, вместо неё, при выполнении такси будут подставляться значения указанные в `loop:`.

Массивы также можно использовать с несколькими параметрами:

```yaml
    - name: add test users
      ansible.builtin.user:
        name: "{{ item.name }}"
        state: present
        groups: "{{ item.groups }}"
      loop:
        - { name: testuser1, groups: wheel }
        - { name: testuser2, groups: root }
```

## Условия

Ansible позволяет выполнять таски при определённых условиях. Для этого используется ключ `when` с выражением на питоне. В них можно напрямую использовать переменные и фактры, без шаблонов:

```yaml
- hosts: all
  task:
    - name: shutdown Debian flavored system
      ansible.builtin.command: /sbin/shutdown -t now
      when: ansible_os_family == "Debian"

    - name: shutdown CentOs 6 and 7 system
      ansible.builtin.command: /sbin/shutdown -t now
      when: ansible_distribution == "CentOS" and
            (ansible_distribution_major_version == "6" or ansible_distribution_major_version == "7")
```

Использование переменных:

```yaml
- hosts: all
  vars:
    epic: true
  task:
    - ansible.builtin.shell: echo "This certainly is epic!"
      when: epic
    - ansible.builtin.shell: echo "This certainly isn't epic!"
      when: not epic
```

Использование в циклах:

```yaml
- hosts: all
  task:
    - ansible.builtin.command: echo {{ item }}
      loop: [0, 2, 4, 6, 8, 10]
      when: item > 5
```

Здесь проихсодит проверка, что значение больше 5.

## Фильтры

Шаблонизатор Jinja2 поддерживает фильтры - специальные функции,  которые применяются к переменным и производят с ними какие-то изменения. Соответственно, и в Ansible этот механизм тоже используется: 

```yaml
- hosts: all
  gather_facts: no
  vars:
    numbers: [3, 2, 1, 3, 2]
  tasks:
    - name: get min number
      ansible.builtin.debug: msg={{ numbers | min }}
    - name: get max number
      ansible.builtin.debug: msg={{ numbers | max }}
    - name: get unique values
      ansible.builtin.debug: var={{ item }}
      loop: "{{ numbers | unique }}"
    - name: get random value
      ansible.builtin.debug: msg={{ ['a', 'b', 'c'] | random }}
    - ansible.builtin.debug: msg={{ '192.0.2.1/24' | ipaddr('address') }}
    - ansible.builtin.debug: msg={{ 'test1' | hash('sha1') }}
    - ansible.builtin.debug: msg={{ path | basename }}
    - ansible.builtin.debug: msg={{ path | dirname }}
    - ansible.builtin.debug: msg={{ "~/Movies" | expanduser }}
```

Здесь применяются фльтры к списку чисел:

- `min` находит минимальное значение;
- `max` находит максимальное значение;
- `unique` находит уникальные значения (результатом является список, поэтому для вывода на печать используем цикл);
- `random` позволяет выбрать случайное значение из списка;
- `ipaddr` позволяет извлекать из ip-адреса  различные части. Для его работы возможно придется установить  дополнительный Python-пакет. В документации об этом написано;
- `hash` создает хэш;
-  `basename` и `dirname` работают с путями. С помощью них можно выделить имя файла и путь к директории;
- `expanduser` раскрывает тильды и делает подстановку.

[Документация по фильтрам](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#using-filters-to-manipulate-data)

## Регистрация результата

В ansible существует механизм _регстрация результата_, который позволяет использовать результат выполнения одной такси в другой.

```yaml
- hosts: all
  gather_facts: no
  tasks:
    - ansible.builtin.shell: ls /Users
      register: home_dirs
    - name: add home dirs to cron
      ansible.builtin.cron:
        name: "backup_dirs"
        minute: "0"
        hour: "5,2"
        job: "backup /home/{{ item }}"
      with_items: home_dirs.stdout_lines
    - ansible.builtin.debug:
        var: home_dirs.stdout_lines
```

С помощью ключа `register`, можно записать результат работы модуля `shell` в переменную, в данном случае в `home_dirs`. Затем эту переменную можно использовать в последующих тасках.

В нашем случае `home_dirs` будет содержать примерно такие данные:

```bash
ansible-playbook playbook.yml -i inventory.ini
TASK: [print home_dirs variable] ****************
ok: [localhost] => {
    "var": {
        "home_dirs": {
            "changed": true,
            "cmd": "ls /Users",
            "delta": "0:00:00.011196",
            "end": "2020-08-11 15:20:12.739441",
            "failed": false,
            "rc": 0,
            "start": "2020-08-11 15:20:12.728245",
            "stderr": "",
            "stderr_lines": [],
            "stdout": "Guest\nShared\nkirill"
            "stdout_lines": [
                "Guest",
                "Shared",
                "kirill"
            ]
        }
    }
}
```

`home_dirs` — хэш, внутри него содержится информация о таске, самое главное здесь это `stderr` и `stdout`.  Причём `stdout` представлен в двух форматах: в виде строки и списка.

Ещё пример:

```yaml
- hosts: all
  gather_facts: no
  tasks:
    - ansible.builtin.command: 'false'
      register: result
      ignore_errors: yes
    - ansible.builtin.command: echo 'ehu'
      when: not result.failed
    - ansible.builtin.command: uptime
      when: result.failed
```

Здесь включено игнорирования ошибок для команды `ansible.builtin.command: 'false'` , чтобы ошибка в ней не приводила к остановке выполнения плейбука, далее проверяется результат этой команды и в зависимости от успешности выполения этой команды выполянются остальные команды.

## Включения

В плейбук можно включать другие файлы, для этого существуют модули:

- [include_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html)
- [import_playbook](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_playbook_module.html)
- [import_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_tasks_module.html)

Модули `include_tasks` и `import_tasks` очень схожи, но ansible обрабатывает их по-разному. Для статических задач используется `import*`, а для динамических `include*`. Подробней про эти две модели можно узнать из официальной документации:

- [Including and Import](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_includes.html)
- [Creating Reusable Playbooks](https://docs.ansible.com/ansible/latest/playbooks_reuse.html)

`include` накладывает ряд ограничений в связи с его динамической природой, поэтому на практике обычно используется `import_tasks` или `import_playbook`.

## Роли

Ansible выделяет повторяющиеся вещи в роли. Роли выкладываются в общий [каталог](https://galaxy.ansible.com/home), где можно найти готовое решение по установке и настройке.

Роль прдеставляет из себя набор задач или обработчик переменных, файлав и других аретфакторв, которые располагаются и подключаются как единое целое к плейбуку.

Пример: устновка nginx, есть [официальная роль для его установки](https://galaxy.ansible.com/nginxinc/nginx).

Установка роли выполняется так:

```bash
ansible-galaxy install nginxinc.nginx
```

Имя роли состоит из неймспейса (nginxinc) и именем самой роли (ngnix).

Подключение роли:

```yaml
- hosts: all
  roles:
    - role: nginxinc.nginx
  tasks: ...
```

Ansible выполняет роли до `tasks`, независимо от порядка ключей в плейбуке. Если требуется выполнить какие-то действие до роли то есть два варианта:

* Добавить задачи в `pre_tasks`:

```yaml
- hosts: all
  pre_tasks: # Выполняются до ролей
    # Тут список задач
    - name: Какая-то задача
      ansible.builtin.shell: # Делаем что-нибудь
  roles:
    - role: nginxinc.nginx
    - role: # какая-нибудь другая роль
  tasks: # Выполняются после ролей
    # Тут список задач
```

* Добавить роли через `import_role`:

```yaml
- hosts: all
  tasks:
    - name: Какая-то задача
      ansible.builtin.shell: # Делаем что-нибудь

    - name: Ставим nginx через роль
      import_role:
        name: nginxinc.nginx

    - name: Какая-то задача
      ansible.builtin.shell: # Делаем что-нибудь
```

### Конфигурация ролей

Управлять поведением роли можно с помощью перепеменных.

Переменные можно найти в директории _defaults_ в репозитории роли, например в nginx:

```yaml
# Enable NGINX and NGINX modules.
# Variables for these options can be found below.
# Default is true.
nginx_enable: true

# Print NGINX configuration file to terminal after executing playbook.
nginx_debug_output: false

# Install NGINX Dynamic Modules.
# You can select any of the dynamic modules listed below. Beware of NGINX Plus only dynamic modules (these are marked).
# Format is list with either the dynamic module name or a dictionary (see njs for an example).
# When using a dictionary, the default value for state is present, and for version it's nginx_version if specified.
# Default is an empty list (no dynamic modules are installed).
nginx_modules: []
  # - auth-spnego  # NGINX Plus
  # - brotli  # NGINX Plus
  # - cookie-flag  # NGINX Plus
  # - encrypted-session  # NGINX Plus
  # - geoip
```

Переопределяются переменные так:

```yaml

# roles
- hosts: all
  roles:
    - role: nginxinc.nginx
      vars:
        nginx_debug_output: true
        nginx_modules:
          - geoip

# import_role
- hosts: all
  tasks:
    - import_role:
        name: nginxinc.nginx
      vars:
        nginx_debug_output: true
        nginx_modules:
          - geoip
```

### Автоматическая установка

Роли можно устанавливать автоматически, для этого создаётся файл _requirements.yml_, в который добавляется список нужных ролей:

```yaml
roles:
  # Install a role from Ansible Galaxy.
  - name: geerlingguy.java
    version: 1.9.6
```

Установка выполняется командой:

```bash
ansible-galaxy install -r requirements.yml
```

## Коллекции

Коллекции — формат распостранения связанного набора плейбуков, ролей, модулей и плавунов. Все встроенные модули лежат [здесь](https://github.com/ansible-collections).

Список установеленных коллекций можно посмотреть так:

```bash
ansible-galaxy collection list
```

Коллекция *ansible.builtin* здесь не указывается, т.к. она встроена в ядро Ansible.

Коллекции именуются так же как роли: _неймспейс.название_.

В самой коллекции содержатся модули:

```yaml
- hosts: all
  tasks:
    - name: Создание новой базы данных
      community.postgresql.postgresql_db:
        name: hexlet-development

    - name: Создание дампа существующей базы данных
      community.postgresql.postgresql_db:
        name: hexlet-production
        state: dump
        target: /tmp/hexlet.production.sql
```

Если коллекция не входит в поставку её можно установить командой:

```bash
ansible-galaxy collection install nginxinc.nginx_core
```

[Пример с конфигурацией](https://github.com/nginxinc/ansible-collection-nginx/blob/main/playbooks/deploy-nginx-web-server.yml).

### Автоматическая установка

Как и роли, коллекции можно устанавливать автоматически, в файле *requirements.yml*:

```yaml
collections:
  # Install a collection from Ansible Galaxy
  - name: geerlingguy.php_roles
    version: 0.9.3
```

Затем запускается установка:

```bash
ansible-galaxy install -r requirements.yml
```

## Ansible Vault

Ansible Vault — механизм шифрования переменных и файлов, который помогает спрятать секретные данные (пароли, ключи и т.п.).

У Ansible Vault есть множество вариантов, использования, о них можно почитать в [документации](https://docs.ansible.com/ansible/latest/vault_guide/index.html). Ниже один из способов использования.

1. Создать директорию _group_vars/all_ в директории, откуда запускается ansible. Yaml-файлы, внутри этой директории автоматически подгружаются во время выполнения плейбуков.
2. Создать файл _group_vars/all/vault.yml_, в нём будут храниться зашифрованные переменные.
3. Добавить туда переменную с любым именем и нужным значением.

Для шифрования нужно выполнить команду:

```bash
ansible-vault encrypt group_vars/all/vault.yml
```

Перед шифрованием ansible запроси пароль, который нужно запомнить. После установки пароля файл будет зашифрован. Его содержимое будет вылядеть примерно так:

```
$ANSIBLE_VAULT;1.1;AES256
34613437623263653438316466656437306631343062656565373638393736653966313830393232
6135363366393766333539653237353338316436663636610a623837646561616234393261613836
65376434396536383563626634346365663364313661313766363065616638383035636130313162
3261663433663839660a636239386136373864626665656434643530656439323836653063666336
6664
```

Для того, чтобы использовать зашифрованные данные во время запуска ansible нужно указать опцию ` --ask-vault-pass`:

```bash
ansible-playbook -i inventory --ask-vault-pass playbook.yml
```

Чтобы постоянно не вводить пароль его можно положить в какой-нибудь файл. И указать к нему путь в опции `--vault-password-file`:

```bash
ansible-playbook --vault-password-file /path/to/my/vault-password-file ...
```

Для редактирования файла-хранилища нужно использовать команду:

```bash
ansible-vault edit group_vars/all/vault.yml
```
