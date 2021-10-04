# Where

Where предназначено для фильтрации нежелательных данных в выборке.

```SQL
SELECT emp_id, fname, lname, start_date, title
FROM employee
WHERE title = 'Head Teller';
```

Фильтры в блоке where можно комбинировать с помощью `and`, `or` и `not`. 
Условия так-же можно группировать с помощью скобок.
