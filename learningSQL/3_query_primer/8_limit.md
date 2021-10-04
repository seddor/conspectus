# Limit

Для задания лимита выборки используется `limit`:
```SQL
SELECT * FROM users LIMIT 10
```
выведет первые 10 записей таблицы users.

## Смещения

```SQL
SELECT * FROM users LIMIT 5, 10
```
первые 10 записей после 5й

Аналогичный запрос:
```SQL
SELECT * FROM users LIMIT 10 OFFSET 5
```
