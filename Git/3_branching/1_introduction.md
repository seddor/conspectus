# О ветвлении в двух словах

Ветка — легко премещаемый указатель на один из коммитов, все коммиты хранят указатель на коммит предшествующий ему.

## Создание 

```bash
git branch <name>
```
В результате будет создан новый указатель на текущий коммит. Переключение на ветку не происходит.

Текущая ветка определяется с помощью специального указателя `HEAD` 

```bash
git log --oneline --decorate
```
Покажет на какие коммиты указывают указатели(ветки).

## Переключение

```bash
git checkout <branch_name>
```
В результате `HEAD` переместится туда, куда указывает `<branch_name>`

![линейное ветвление](advance-testing.png)

На картинке есть ветка `master` и `testing`, как видно текущая ветка `testting`(именно на неё указывает `HEAD`). `Testing` опережает `master` на 1 коммит.

Если вернуться в `master` и сделать там ещё один коммит, то тогда ситуация будет как на картинке ниже:
будут два независимых, изолированных изменения.

![многоуровнвое ветвление](advance-master.png)

```bash
git log --oneline --decorate --graph --all
```
покажет историю коммитов и где находятся указатели веток и как ветвилась история проекта.