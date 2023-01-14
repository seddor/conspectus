# Создание базы данных

```sql
CREATE DATABASE db_name CHARACTER SET utf8 COLLATE utf8_general_ci;
```

Создаст базу с указаным чарсетом utf8 и методом сравнения utf8_general_ci.

>В MySQl utf8 является алиасом к `utf8mb3`, на это стоит обратить внимание если нужна расширенная поддержка юникода, например современные эмойдзи, тогда лучше использовать `utf8mb4` и метод сравнения `utf8mb4_unicode_ci`

```SQL
grant all privileges on bank.* to 'username'@'localhost' identified by 'password';
```
Создать пользователя usernamre со всеми правами на БД bank.

```SQL
use bank;
```
Использовать БД bank в текущем сеансе.

```SQL
SELECT now();
```
now() — функция mysql выводящая текущее время.