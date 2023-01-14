# Join

Для запроса данных из нескольких связаных таблиц используются джоины.

## Декартово произведение

Если попробовать использовать join без указания типа и полей, по которым производится соединение то будет произведено декартово произведение всех строк одной таблицы на другую:
```SQL
SELECT e.fname, e.lname, d.name FROM employee e JOIN department d;
```
Такой тип джоина так-же известен как `cross join`.

## Inner joins

Для указания полей для соединения используется ON:
```SQL
SELECT e.fname, e.lname, d.name
FROM employee e JOIN department d
ON e.dept_id = d.dept_id;
```
В таком случае будет использоватся `inner join`, в выборке будут только строки имеющие соотвия по указаному полю в обоих таблцах. `inner join` используется по умолчанию, при указании столбцов для соединения. Эта запись идентична предыдущей:
```SQL
SELECT e.fname, e.lname, d.name
FROM employee e INNER JOIN department d
ON e.dept_id = d.dept_id;
```
Если столбец имеет одинаковое имя в обоих таблицах то можно вместо `ON` использовать `USING (field)`.

## Соединения 3х и более таблиц

В принципе всё аналогично двухтабличному соединению:
```SQL
SELECT a.account_id, c.fed_id, e.fname, e.lname
FROM account a INNER JOIN customer c
ON a.cust_id = c.cust_id
INNER JOIN employee e
ON a.open_emp_id = e.emp_id
```

### Порядок соединения

Обычно порядок в котором джойнятся таблицы не имеет особой роли, т.к. СУБД сама выберет в каком порядке лучше джойнить таблицы.
Однако, если нужно всё таки задать порядок, то СУБД можно задать режим в котором соединения будет происходить в заданом в запросе порядке, каждом СУБД свои способы это сделать, для MySQL нужно использовать `STRAIGHT_JOIN`:
```SQL
SELECT STRAIGHT_JOIN a.account_id, c.fed_id, e.fname, e.lname
FROM customer c INNER JOIN account a
ON a.cust_id = c.cust_id
INNER JOIN employee e
ON a.open_emp_id = e.emp_id
WHERE c.cust_type_cd = 'B';
```

## Использование позапросов

Помимо соединения таблиц с таблицами, можно так-же соединять таблицы с подзапросами:
```SQL
SELECT a.account_id, a.cust_id, a.open_date, a.product_cd
FROM account a INNER JOIN
    (SELECT emp_id, assigned_branch_id
    FROM employee
    WHERE start_date < '2007-01-01'
    AND (title = 'Teller' OR title = 'Head Teller')) e
ON a.open_emp_id = e.emp_id
INNER JOIN
    (SELECT branch_id
    FROM branch
    WHERE name = 'Woburn Branch') b
ON e.assigned_branch_id = b.branch_id;
```

## Использования одной таблицы несколько раз

Одну таблицу можно приджойнить несколько раз, в таком случае для каждого такого джоина нужно указать свои алиасы:
```SQL
SELECT a.account_id, e.emp_id,
b_a.name open_branch, b_e.name emp_branch
FROM account a INNER JOIN branch b_a
ON a.open_branch_id = b_a.branch_id
INNER JOIN employee e
ON a.open_emp_id = e.emp_id
INNER JOIN branch b_e
ON e.assigned_branch_id = b_e.branch_id
WHERE a.product_cd = 'CHK';
```

## Self-join

Можно делать соединения таблицы с этой же таблицей. Например, если в таблице есть ключ, котовый указывает на запись в этой же таблице:
```SQL
SELECT e.fname, e.lname,
e_mgr.fname mgr_fname, e_mgr.lname mgr_lname
    FROM employee e INNER JOIN employee e_mgr
ON e.superior_emp_id = e_mgr.emp_id;
```

## Equi-join vs. non-equi-joins

* Equi-join — джоины соединяемые по эквивалентным значениям, например по foregien key
* non-equi-joins — джоины по не эквивалентым значениям, например:
```SQL
SELECT e.emp_id, e.fname, e.lname, e.start_date
FROM employee e INNER JOIN product p
    ON e.start_date >= p.date_offered
    AND e.start_date <= p.date_retired
```
Также существует `self-non-equi-join` для джоина таблицы с ней же, по не равным значениям, например если нужно составить пары строк:
```SQL
SELECT e1.fname, e1.lname, e2.fname, e2.lname
FROM employee e1 INNER JOIN employee e2
    ON e1.emp_id != e2.emp_id
```
В этой выборке будут обратные пары, чтобы избежать этого:
```SQL
SELECT e1.fname, e1.lname, e2.fname, e2.lname
FROM employee e1 INNER JOIN employee e2
    ON e1.emp_id < e2.emp_id
```
