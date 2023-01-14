# From

## Таблицы

Таблица в БД, это набор взаимосвязанных строк, таблицы могут быть нескольких видов:

* Постоянные таблицы(созданные командой `create table`)
* Временные таблицы(собраные из подзапроса во время выполнения текущего запроса)
* Виртуальные таблцицы(созданиые с помощью `create view`)

### Подразпросы

Подзапрос — запрос содержащийся в другом запросе. Подзапросы включаются в круглые скобки и могут располагаются внути блока select, однако если поместить подзапрос внутри from, то он будет выполнять роль времнной таблицы, способной взаимодействовать с другими таблицами из блока from:
```SQL
SELECT e.emp_id, e.fname, e.lname 
FROM (SELECT emp_id, fname, lname, start_date, title FROM employee) e;
```

## Представляния (Views)

Представление — это запрос хранящийся в словаре данных. Он похож на постоянные таблицы, оданако с представлением несвязаны никакие данные, при выполнении запроса к нему запрос сливается с описанием представления тем самым формируя запрос.

Создание представления:
```SQL
CREATE VIEW employee_vw AS
SELECT emp_id, fname, lname,
    YEAR(start_date) start_year
FROM employee;
```
После создания в БД не создаются никакие данные, сервер просто сохряняет этот запрос для дальнеёшего использования:
```SQL
SELECT emp_id, start_year
FROM employee_vw;
```
Аналогично такому запросу:
```SQL
SELECT emp_id, YEAR(start_date) as start_year
FROM employee;
```

## Связи таблиц

Указать несколько постоянных таблиц в select можно с помощию джоинов.
```SQL
SELECT employee.emp_id, employee.fname, employee.lname, department.name AS dept_name
FROM employee INNER JOIN department
    ON employee.dept_id = department.dept_id;
```
Будут выведены все работники у которых есть депармент а также названия департементов из таблицы department.

## Алиасы для таблиц

Указать столбец из другой таблицы можно несколькими способами:
* использовать подное имя таблиц и имя столбца: `department.dept_id`
* присвоить таблице алиса и использовать его:
```SQL
SELECT e.emp_id, e.fname, e.lname,
d.name dept_name
FROM employee e INNER JOIN department d
ON e.dept_id = d.dept_id;
```
