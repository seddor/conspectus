# mySQL

## Основные команды

```SQL
#создание БД
CREATE DATABASE db_name;
#использовать БД как текущую
USE db_name;
#показать структуру таблицы
DESCRIBE tаble_name;
#создать таблицу
CREATE TABLE pet (name VARCHAR(20);
#удалить таблицу
DROP TABLE forums;
#удалить БД
DROP DATABASE database_name;
#удаление записей
DELETE FROM table_name [WHERE definition]
```

## Изменение структуры таблицы

```SQL
ALTER TABLE table_name alter_spec
```
alter_spec:

- `ADD create_definition [FIRST|TABLE column_name]`

Добавление нового столбца create_definition. create_definition представляет собой название нового столбца и его тип. Конструкция FIRST добавляет новый столбец перед столбцом column_name. Конструкция AFTER добавляет новый столбец после столбца column_name. Если место добавления не указано, по умолчанию столбец добавляется в конец таблицы.

- `ADD INDEX [index_name] (index_col_name,...)`

Добавление индекса index_name для столбца index_col_name. Если имя индекса index_name не указывается, ему присваивается имя совпадающее с именем столбца index_col_name.

- `CHANGE old_col_name new_col_name type`

Изменение столбца с именем old_col_name на столбец с именем new_col_name и типом type.

- `DROP col_name`

Удаление столбца с именем col_name.

- `DROP PRIMARY KEY`

Удаление первичного ключа таблицы.

- `DROP INDEX index_name`

Удаление индекса index_name.

```SQL
ALTER TABLE forums ADD test int(10) AFTER name;
ALTER TABLE forums CHANGE test new_test text;
ALTER TABLE forums CHANGE new_test new_test int(5) not null;
ALTER TABLE forums DROP new_test;
```

## Вставка новых записей

```SQL
INSERT INTO table_name VALUES (values,…)
```
После оператора VALUES в скобках через запятую перечисляются значения соответствующих полей таблицы в соответствии с их типами.

```SQL
INSERT INTO `service_mobility_variants` (`id`, `name`) VALUES
(1, 'similique'),
(2, 'et'),
(3, 'voluptatem'),
(4, 'quisquam'),
(5, 'sit'),
(6, 'fugiat'),
(7, 'non'),
(8, 'optio'),
(9, 'aut'),
(10, 'aspernatur'),
(11, 'molestias'),
(12, 'excepturi'),
(13, 'voluptas'),
(14, 'tenetur'),
(15, 'deleniti');

INSERT INTO `service_mobility_variants` VALUES (NULL, 'foobar');
```

## Выборка

```SQL
SELECT column,...
[FROM table WHERE definition]
[ORDER BY col_name [ASC | DESC], ...]
[LIMIT [offset], rows]
```

## Обновление записей

```SQL
UPDATE table
    SET col_name1=expr1 [, col_name2=expr2 ...]
    [WHERE definition]
    [LIMIT rows]
```

## Show

```SQL
#показать все БД
SHOW DATABASES;
#Показать столбцы таблицы
SHOW FIELDS FROM authors;
#Показать индексы
SHOW INDEX FROM authors;
```
