# Типы подзапросов

## Несвязанные подзапросы

Подзапрос может выполнятся самостоятельно и не использует ничего из содержащего выражения. 

### Скалярные запросы

Подзапросы возвращающие одну строку с одним столбцом являются скалярными, их результаты можно использовать в условиях операторов фильтрации(=, <>, <, >, <=, >=).

Если запрос который должен быть скалярным вернёт более одной строки, то будет ошибка.

## Подзапросы возвращающие несколько строк и один столбец

Есть 4 оператора позволяющих строить условия с такими подзапросами:

* IN

Проверяет наличие какого-либо значения в наборе, в данном случае набором является резлуьтат подзапроса:
```SQL
SELECT emp_id, fname, lname, title
    FROM employee
    WHERE emp_id IN (SELECT superior_emp_id
    FROM employee);
```

Так-же можно использовать инверсию, для проверки отсутвия в наборе: `NOT IN`.

* ALL

ALL позволяет производить сравнения одиночного значения с каждым значением набора. Для условий с ALL нужно так-же использовать один из операторов: =, <>, <, >, <=, >=. 

```SQL
SELECT emp_id, fname, lname, title
    FROM employee
    HERE emp_id <> ALL (SELECT superior_emp_id
    ROM employee
    HERE superior_emp_id IS NOT NULL)
```

>Сравнивать значения с набором значений с помощью операторов `not in` или `<> all` нужно аккуратно, убедившись, что в наборе нет значения null. Сервер приравнивает значение из левой части выражения к каждому члену набора, и любая попытка приравнять значение к null дает в результате unknown. Таким образом, следующий запрос возвратит пустой набор:
```SQL
SELECT emp_id, fname, lname, title
    FROM employee
    WHERE emp_id NOT IN (1, 2, NULL);
```
* ANY

Аналогичен оператору `ALL`, оператор `ANY` сравнивает все значение с элементами набора, только для `ANY` нужно чтобы хотя-бы одно из значений оказалось истино. 

Например нужно найти счета, остаток на которых больше, чем на любом из счетов клиента Frank Tucker:
```SQL
SELECT account_id, cust_id, product_cd, avail_balance
    FROM account
    WHERE avail_balance > ANY (SELECT a.avail_balance
    FROM account a INNER JOIN individual i
        ON a.cust_id = i.cust_id
    WHERE i.fname = 'Frank' AND i.lname = 'Tucker');
```

> = any и in эквивалентны.

## Связанные подзапросы

Связанный подзапрос зависит от содержащего его запроса. 
```SQL
SELECT c.cust_id, c.cust_type_cd, c.city
    FROM customer c
    WHERE 2 = (SELECT COUNT(*)
    FROM account a
    WHERE a.cust_id = c.cust_id);
```

Здесь подзапрос ссылается но основной запрос ссылкой c.cust_id.

В данном случае основной запрос извлекает всех клиентов из customer и выполяняет для каждого подзапрос, если результат подзапроса равен 2, то выполняется условие фильрации.

### Оператор exists

Exists проверяет наличие связи в запросе, при этом количества связей не важно:
```SQL
SELECT a.account_id, a.product_cd, a.cust_id, a.avail_balance
FROM account a
WHERE EXISTS (SELECT 1
    FROM transaction t
    WHERE t.account_id = a.account_id
        AND t.txn_date = '2005-01-22');
```
Запрос находит все счета, для которых транзакция была выполнена в опеределёныый день, без учёта количества транзацкий.

Для поиска подзапросов не взовращающих ничего можно использовать инверсию:
```SQL
SELECT a.account_id, a.product_cd, a.cust_id
    FROM account a
    WHERE NOT EXISTS (SELECT 1
    FROM business b
    WHERE b.cust_id = a.cust_id);
```
