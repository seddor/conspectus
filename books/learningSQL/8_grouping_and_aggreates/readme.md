# Группировка

Групировка позволяет объединять данные по одинаковым значениям поля:

```SQL
SELECT open_emp_id
FROM account
GROUP BY open_emp_id;
```

## Агрегатные функции

* Count()

При использовании группировок можно использовать агрегатные функции, например `count()` подсчитывающую количество записей:
```SQL
SELECT open_emp_id, COUNT(*) how_many
FROM account
```

Для использования агрегатных функций в фильтрации нужно использовать `HAVING`:
```SQL
SELECT open_emp_id, COUNT(*) how_many
FROM account
GROUP BY open_emp_id
HAVING COUNT(*) > 4;
```

* Max() — максимальное значение в группе
* Min() — минимальное значение в группе
* Avg() — среднее значение в группе
* Sum() — сумма значений в группе

### Подсчёт уникальных значений

Если нужно подсчитать количество ункальных записей:
```SQL
SELECT COUNT(DISTINCT open_emp_id) FROM account
```

### Работа с null 

Функиции `sum()`, `max()`, `min()`, `avg()` игнорируют null при подсчёте.
`Count(*)` — будет считать null как обычную строку.
`Count(val)` — будет игнорировать null, т.к. считает значения.

## Форимрование групп

### Группировка по одному столбцу

```SQL
SELECT product_cd, SUM(avail_balance) prod_balance
FROM account
GROUP BY product_cd;
```

### Группировка по нескольким столбцам

```SQL
SELECT product_cd, open_branch_id,
SUM(avail_balance) tot_balance
FROM account
GROUP BY product_cd, open_branch_id;
```

Посчитант общий остаток для каждого обноруженого сочетания значений product_cd и open_branch_id.

### Группировка посредствои выражений

```SQL
SELECT EXTRACT(YEAR FROM start_date) year,
    COUNT(*) how_many
    FROM employee
    GROUP BY EXTRACT(YEAR FROM start_date);
```

### Форимрование обобщений

Если нужно помимо суммы для каждого сочетания значений получить так-же суммирующее для каждого типа, можно использовать обобщения:
```SQL
SELECT product_cd, open_branch_id,
    SUM(avail_balance) tot_balance
    FROM account
    GROUP BY product_cd, open_branch_id WITH ROLLUP;
```
Результаты будут аналогичны тем, что в Группировка по нескольким столбцам + будут значения с null в `open_branch_id` — это обобщёные значения, и в конце будет общая сумма по всем записям.

# Примечания для MySQL с версии 5.7

Начиная с этой версии MySQL используется [ONLY_FULL_GROUP_BY](https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_only_full_group_by) режим для группировок, это значит, что группировать можно только однозначо определяемые данные в группе и данные с агрегированные с помощью функций.

Раньше при группировке по умолчанию для необределёных данных бралось первое найденое значение при создании группы, теперь в таком случае будет ошибка. 
Если всё таки нужно исползьовать значение, (например, известно, что оно одинаково для всей группы), то нужно использовать агрегатную функцию `ANY_VALUE()`:
```SQL
select login, ANY_VALUE(`dealer_id`) as dealer_id, CAST(created_at as DATE) as date, count(*) as count 
    from admin_successful_login
    group by login, date;
```
