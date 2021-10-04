# Перекрёстные соединения (Cross joins)

Перекрёстное соединение задаётся так:
```SQL
SELECT pt.name, p.product_cd, p.name
    FROM product p CROSS JOIN product_type pt;
```
В результирующей выборку будет сочетание каждой записи из левой стаблицы с правой.

