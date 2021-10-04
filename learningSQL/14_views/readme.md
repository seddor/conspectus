# Представления (Views)

Представления это механизм для представления данных, он подобен обычным таблицам, однако в отличии от них он не хранит данные, а агригирует их из других таблиц. По сути это сохранёный запрос.

```SQL
CREATE VIEW customer_vw(
    cust_id,
    fed_id,
    cust_type_cd,
    address,
    city,
    state,
    zipcode
    )
AS SELECT 
    cust_id, 
    concat('ends in ', substr(fed_id 8, 4)) fed_id,
    cust_type_cd,
    address,
    city,
    state,
    postal_code
FROM customer;
```
После этого с представлением можно запрашивать:
```SQL
SELECT cust_id, fed_id, cust_type_cd FROM customer_vw;
```

## Измения с помощью представлений

Представления так-же можно использовать в запросах на измения данных:
```SQL
PDATE customer_vw
    SET city = 'Woooburn'
    WHERE city = 'Woburn';
```
Изменять можно только обычные столбцы таблиц, если столбец получен из выражений(например, как fed_id) то его изменить не получится.

Так-же в представлениях можно использовать `INSERT`:
```SQL
INSERT INTO business_customer_vw
    (cust_id, fed_id, address, city, state, postal_code)
    VALUES (99, '04-9999999', '99 Main St.', 'Peabody', 'MA', '01975');
```

Стоит обрать внимание, что для сложных представлений из нескольких таблиц, одновременно можно изменять значения только той части представления, чвто принадлежит одной таблице.
