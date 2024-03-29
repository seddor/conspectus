# Просмотр истории коммитов

Основной инструмент для просмотра истории в git, это команда:

```bash
git log
```

пример вывода:

```
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit
```

Коммиты отображаются в обратном хронологическом порядке. 


* `-p` — показывает диф для кажого коммита
* `--stat` — показывает сокращённу сводку для каждого коммита(количество изменённых файлов, добавлений, удалений и т.п.)
* `--pretty=format` — изменяет формат вывода. Формат можно выбрать из готовых констант:
    * oneline — вывод в одну линию
    * short — чуть сокращёная версия стандартного вывода
    * full
    * fuller
Или можно воспользоваться опциями для построения кастоного формата:
    * `%H` — Хеш коммита
    * `%h` — Сокращенный хеш коммита
    * `%T` — Хеш дерева
    * `%t` — Сокращенный хеш дерева
    * `%P` — Хеш родителей
    * `%p` — Сокращенный хеш родителей
    * `%an` — Имя автора
    * `%ae` — Электронная почта автора
    * `%ad` — Дата автора (формат даты можно задать опцией --date=option)
    * `%ar` — Относительная дата автора
    * `%cn` — Имя коммитера
    * `%ce` — Электронная почта коммитера
    * `%cd` — Дата коммитера
    * `%cr` — Относительная дата коммитера
    * `%s`— Содержание

Разница между автором и коммитером: Автор — это человек, изначально сделавший работу, а коммитер — это человек, который последним применил эту работу. 
>Другими словами, если вы создадите патч для какого-то проекта, а один из основных членов команды этого проекта применит этот патч, вы оба получите статус участника – вы как автор и основной член команды как коммитер.

* `--graph`­­ — отображет граф, показывающий текущую ветку и историю мерджей.
* `--relative-date` — отображает дату в относительном формате, например: 2 weeks ago.
* `--abbrev-commit` — скорущёный хеш комитов.
* `--name-status` — показывает список добавленных/изменённых/удалёных файлов.
* `--shortstat` — Отображает только строку с количеством изменений/вставок/удалений для команды `--stat`.

## Ограничения вывода

* `-(число)` — здесь можно ввести любое число, которое будет обозначать количество коммитов, которы надо вывести, например `-2` покажет последние 2 коммита.
* `--since` ­— отображет коммиты сделаные за начиная с определёного периода, можно задать разные значения:
    * 2.weeks
    * "2008-01-15"
    * "2 years 1 day 3 minutes ago"
* `--until` — аналогично `--since`, только до определённого времени.
* `--author` — фильтр по автору.
* `--grep` — искать по сообщениям коммитов.
>Имейте ввиду, что если вы хотите фильтровать коммиты по автору и ключевым словам одновременно, вам нужно также добавить --all-match. В противном случае, команда отфильтрует вывод по одному из двух критериев.
* `-S` — поиск по контенту коммитов.
