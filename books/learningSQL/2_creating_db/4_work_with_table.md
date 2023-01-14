# Заполнение и изменение таблци

## Добавление данных 

Используется команда `INSERT INTO`:
```SQL
INSERT INTO person
    (person_id, first_name, last_name, gender, birth_date)
    VALUES (null, 'John', 'Doe', 'M', '1984-05-27');
```

## Изменение 

Если нужно изменить колонку таблицы:
```SQL
ALTER TABLE person MODIFY person_id SMALLINT UNSIGNED AUTO_INCREMENT;
```

Для измения данных в таблицах:
```SQL
UPDATE person
    SET 
     first_name = 'foo',
     last_name = 'bar'
    WHERE person_id = 2;
```

## Удаление

```SQL
DELETE FROM person WHERE person_id = 2;
```

## Генерация XML

Mysql может генерировать xml для селектов, для этого нужно запустить клиент с параметров `--xml`
```SQL
mysql -u <user> -p --xml <db_name>
```
Результаты селектов будут примерно такие:
```xml
<?xml version="1.0"?>

<resultset statement="select * from person;" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <row>
	<field name="person_id">1</field>
	<field name="first_name">John</field>
	<field name="last_name">Doe</field>
	<field name="gender">M</field>
	<field name="birth_date">1984-05-27</field>
  </row>
</resultset>
```
