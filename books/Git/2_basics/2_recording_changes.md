# Запись изменений в репозиторий

Все файлы в рабочей директории проекта находятся в двух состояниях 

* не отслеживаемые — файлы не отслеживаются git, или новые файлы которые не были добавлены под версионый контроль или игнорируемые файлы
* отслеживаемые — файлы под версионым контролем, они могут находится в 3 состояниях(зафиксированые, изменёные и подготовленные к коммиту).

![Жизеннный цикл файлов](lifecycle.png)

## Определения состояния файлов

Основная команда используемая для получения текущего состояния рабочей директории:

```bash
get status 
```

Она показывает положения указателя HEAD(см. ветвление), обычно это название ветки.

Показывает есть ли новые/изменённые/неотслеживаемые файлы и их состояние, если есть.

Также отображется информация о состоянии текущей ветки на удалёном сервере, если такая информация есть.

* Параметр `-s` или `--short` показывает сокрашёный вариант состояния, в котором показывается только инфа по файлам.

## Отслеживание файлов

Команда для добавление файла под версионный контроль:

```bash
git add <files_template>
```

команда принимает параметром относительный путь к файлу, или каталогу, если указан каталог, то файлы из него будут рекрсивно добавлены.

*Эта же команда используется для занесения в индекс уже отслеживаемых файлов, но изменённых.*

* Параметр `--all` добавляет все файлы в индекс.

## Игнорирование

Для задавания списка игнрорируемых файлов версионным контролем используется файл _.gitignore_. Он содержит в себе перечисления шаблонов имён файлов или директорий, которые нужно игнорировать. 

Синтаксис _.gitignore_

* Пустые строки, а также строки, начинающиеся с #, игнорируются.
* Можно использовать стандартные [glob шаблоны](https://en.wikipedia.org/wiki/Glob_(programming)).
* Можно начать шаблон символом слэша (/) чтобы избежать рекурсии.
* Можно заканчивать шаблон символом слэша (/) для указания каталога.
* Можно инвертировать шаблон, использовав восклицательный знак (!) в качестве первого символа.

>Glob-шаблоны представляют собой упрощённые регулярные выражения, используемые командными интерпретаторами. Символ (*) соответствует 0 или более символам; последовательность [abc] — любому символу из указанных в скобках (в данном примере a, b или c); знак вопроса (?) соответствует одному символу; и квадратные скобки, в которые заключены символы, разделённые дефисом ([0-9]), соответствуют любому символу из интервала (в данном случае от 0 до 9). Вы также можете использовать две звёздочки, чтобы указать на вложенные директории: a/**/z соответствует a/z, a/b/z, a/b/c/z, и так далее.

[Примеры _.gitignore_ для множества популярных языков/фреймворков](https://github.com/github/gitignore)

### Локальное игнорирования файлов 

Файл _.gitignore_ используется для глобального исключения файлов из репозитория, однако если нужно исключить файл локально, то нужно использовать _.git/info/exclude_.
По сути это тот же gitignore только локальный.

## Коммит изменений

Команда создание коммита 

```bash
git commit
```

Созадёт коммит с текущими изменениями занесёными в индекс, при запуске необходимо ввести сообщение коммита с описанием изменений

* Параметр -m позвоялет указать сообщение коммита сразу
* Параметр -a добавит в коммит все изменённые/новые файлы, а не только те, что находятся в индексе.

## Удаление файлов

Для удаления файла из git необходимо удалить его из отслеживаемых.

```bash
git rm <files_template>
```

Если удалить файл руками, то git будет отображать его состояние как удалёный, не занесённый в индекс.
Для удаление из отслеживания нужно сделать `git rm`

Если файл который нужно удалить уже есть в индексе для коммита, то нужно использовать `git rm -f` для принудительного удлаения. 

* Параметр `--cached` — удаляет файл из версионого контроля, оставляя его физически на диске.
