# Order by

`order by` предназначен для сортировки данных в выборке. 

```SQL
SELECT open_emp_id, product_cd
FROM account
ORDER BY open_emp_id;
```
В качестве сортировки можно задавать несколько условий, тогда сортировка будет производится по каждому условию по очереди, при этом не противоряче предыдущей сортировке:
```SQL
SELECT open_emp_id, product_cd
FROM account
ORDER BY open_emp_id, product_cd;
```

Для задания направления сортировки используется два ключевых слова:
* ASC — ascending, сортировка по возвростанию
* DESC — descending, сортировка по убыванию

Помимо столбцов для сортировки так-же можно использовать различные выражения:
```SQL
SELECT cust_id, cust_type_cd, city, state, fed_id
FROM customer
ORDER BY RIGHT(fed_id, 3);
```
Функция `RIGHT(feild, n)` извлекает последние n символов из столба, order by будет применен к результатам выражения.
