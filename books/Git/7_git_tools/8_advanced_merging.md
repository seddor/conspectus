# Продивинутое слияние

## Конфликты

Прерывания слияния:
```bash
git merge --abort
```
команда откатит рабочий каталог до момента слияния, с ней могут возникнуть проблемы если до слияния в каталоге находились не зафиксированные измения, так что лучше всегда перед мерджем убеждатся, что в рабочей каталоге таких нет.

Если при мердже конфликт в проблемльных символах, то можно использовать:
```bash
git merge -Xignore-all-space/-Xignore-space-change.
```
* `-Xignore-all-space` — игнорирует изсенения в любом количестве существующих пробельных символов.
* `-Xignore-space-change` — игнорирует все измения пробельных символов.


### Ручное слияни файла

В git при конфликте слияния можно получить три версии файла:

* 1 — общий предок для файла
* 2 — версия в текущей ветке
* 3 — состояние из `MERGE_HEAD`, т.е. версия из мердж-ветки

Получить файлы монжо так:
```bash
git show :1:hello.rb > hello.common.rb
git show :2:hello.rb > hello.ours.rb
git show :3:hello.rb > hello.theirs.rb
```

смерджить один файл:
```bash
git merge-file -p \
    hello.ours.rb hello.common.rb hello.theirs.rb > hello.rb
```

Для просмотра разница между сливаемыми измениями:
```bash
git diff
```
* `--ours` — покажет состояние ветки до слияния.
* `--theirs` — покажет разницу результата слияния и текущей ветки.
* `--base` — покажет как файл изменился по сравнению с обоими ветками.
* `-w` — не учитывать измения в проблельных символах при дифе.

### checkout

```bash
git checkout 
```
* `--conflict` — при использовании на файле, заново выкачает файл и заменит маркеры конфликта. Можно указать значение `diff3` и тогда в резултате будет так-же базовая версия файла:
```
#! /usr/bin/env ruby

def hello
<<<<<<< ours
  puts 'hola world'
||||||| base
  puts 'hello world'
=======
  puts 'hello mundo'
>>>>>>> theirs
end

hello()
```
* `--ours` или `--theirs` — позволяет выбрать конкрутную версию, не производя слияния руками.

### История при слиянии

Получения истории коммитов при слиянии:
```bash
git log --oneline --left-right HEAD...MERGE_HEAD
```
можно посмотреть только конфликтующие коммиты:
```bash
git log --oneline --left-right --merge
```
* `-p` — получить только список измений конфликтного файла.

## Отмена слияния

### Исправление ссылок

```bash
git reset --hard HEAD~
```
Подойдёт для отката на один или несколько коммитов. Введёт к измению истории, поэтому опасно если ветка уже используется кем-то ещё. 

### Отмена коммитов

Создать новый коммит, который откртывает измения, сделаные в другом:
```bash
git revert -m 1 HEAD 
```
Здесь `-m 1` указывает какой родитель является основной веткой и должен быть сохранен. 
>Когда вы выполняете слияние в HEAD (git merge topic), новый коммит будет иметь двух родителей: первый из них HEAD (C6), а второй – вершина ветки, которую сливают с текущей (C4). В данном случае, мы хотим отменить все изменения, внесенные слиянием родителя #2 (C4), и сохранить при этом всё содержимое из родителя #1 (C6).

![История после реверта](undomerge-revert.png)

>Новый коммит ^M имеет точно такое же содержимое как C6, таким образом, начиная с нее всё выглядит так, как будто слияние никогда не выполнялось, за тем лишь исключением, что “теперь уже не слитые” коммиты всё также присутствуют в истории HEAD

Поэтому если сново попробовать смерджить отревёрченую ветку, то git скажет, что всё уже слито, т.к. коммиты уже присутвуют в истории. Соотвено, если в `topic` добавить ещё коммитов, а затем слить с `master`, то сольются только новые коммиты, сделаные после ревёрта.

Для того чтобы это исправить нужно "отменить отмену измений":
```bash
git revert ^M
```
После этого можно сделать мердж и получить все нужные коммиты в `master`.

## Другие типы слияний

```bash
git merge
```
* `-Xours` — при возникновении конфликта применять текущую версию из ветки.
* `-Xtheirs` — при возникновении конфликта применять приходящую версию.
