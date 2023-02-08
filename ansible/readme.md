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

