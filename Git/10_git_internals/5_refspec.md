# Спецификации ссылок

При добавлении удалёного репозитория в файл `.git/config` добавляется секция с конфигом для удалёного репозитория, например:
```ini
[remote "origin"]
	url = https://github.com/schacon/simplegit-progit
	fetch = +refs/heads/*:refs/remotes/origin/*
```
в поле fetc описан формат спецификации удалённых ссылок:
* опциональный `+`, означает, что обновления из репозитория надо выполнять, даже если оно не является пермоткой(fast-forward).
* пара `<src>:<dst>`:
    * src — шаблон ссылок в удалённом репозитории
    * dst — шаблон локальных ссылок

По умолчанию git при добавлении удалённого репозитория забирает все ссылки из `refs/heads` на сервере и записывает их в `refs/remotes/orgin` локально.

Если нужно чтобы при обновлении git забирал не все ветки, а только конкретную в шаблоне можно заменить `*` на имя ветки.
Или если нужно так сделать не изменяя шаблон:
```bash
git fetch origin master:refs/remotes/origin/mymaster
```

В конфигурации можно задавать несколько спецификация для обновления веток:
```ini
[remote "origin"]
	url = https://github.com/schacon/simplegit-progit
	fetch = +refs/heads/master:refs/remotes/origin/master
	fetch = +refs/heads/experiment:refs/remotes/origin/experiment
```
Частично задать маску файла нельзя, такая запись ошибочна:
```
fetch = +refs/heads/qa*:refs/remotes/origin/qa*
```

## Спецификация для отправки

Помимо fecth можно так-же задавать настройки для отправки веток на сервер:
```ini
[remote "origin"]
	url = https://github.com/schacon/simplegit-progit
	fetch = +refs/heads/*:refs/remotes/origin/*
	push = refs/heads/master:refs/heads/qa/master
```
Теперь `git push origin` локальный `master` будет по умолчанию залит в удалённюу ветку `qa/master`.

## Удаление ссылок

```bash
git push orign :<branch_name>
```
Т.к. спецификация задаётся в виде `<src>:<dst>`, то пропуская `<src>` git'у указывается, что `<branch_name>` на удалённом сервере нужно сделать пустой, что приводит к её удалению.
