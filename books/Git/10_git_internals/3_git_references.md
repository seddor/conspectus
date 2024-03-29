# Ссылки в Git

Сслыки распологаются в `.git/refs`. По сути ссылка предсавляет из себя просто файл, в котором записан хэш коммита на который она ссылается.

Внутреняя команда для измения ссылок:
```bash
git update-ref refs/heads/master <commit_hash>
```
Ссылки лежащие в `.git/refs/heads` — это ветки.

## HEAD

HEAD тоже является ссылкой, но содержит не сам хэш коммита, а название текущей выбраной ссылки такие ссылки в терменологии git называются символичными.

Для работы с символичными ссылками используется команда:
```bash
git symbolic-ref <name> [value]
```
Если передать только name(например, HEAD), то выведится её текущее значение, если передать ещё value, то у ссылки изменится значение на переданное.
Git позволяет изменять значение символических ссылок только на ссылки из `.git/refs`, иначе будет ошибка.

## Теги

* Легковесные теги представляют из себя простые сслыки на коммиты, хранятся в `.gir/refs/tags`. По сути это ветка, которая не перемещается.
* Анотированный тег — при его создании git создаёт специальный объект, на который будет указывать сслыка.

## Сылки на удалённые ветки

Хранятся в `.git/refs/remotes`, содежрат последнее отправленное значения хеша для веток.
Ссылки на удалённые ветки считаются неизменяемыми, т.е. если попробовать на них переключится git не установит HEAD на такую ссылку, а значит коммит сделать не получится.