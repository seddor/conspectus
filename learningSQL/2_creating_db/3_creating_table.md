# Создание таблиц

```SQL
CREATE TABLE person
    (
        person_id INT UNSIGNED AUTO_INCREMENT,
        first_name VARCHAR(255),
        last_name VARCHAR(255),
        gender ENUM('M','F'),
        birth_date DATE,
        CONSTRAINT pk_person PRIMARY KEY (person_id)
    );
```

Можно добавить проверку на значения для поля gender и принимать только M и F

```SQL
...
gender CHAR(1) CHECK (gender IN ('M','F')),
...
```

или можно использовать специальный тип данных `ENUM`

```SQL
...
gender ENUM('M','F'),
...
```

## Получения сведений

Список тбалиц в базе:
```SQL
show tables;
```

Сведениея о таблице:
```SQL
DESCRIBE <tаble_name>;
```
Или
```SQL
DESC <table_name>;
```

## Foreign key

```
CREATE TABLE favorite_food
(
    person_id INT UNSIGNED,
    food VARCHAR(255),
    CONSTRAINT pk_favorite_food PRIMARY KEY (person_id, food),
    CONSTRAINT fk_fav_food_person_id FOREIGN KEY (person_id) REFERENCES person (person_id)
);
```
Ограничение Foreign key означает, что значение поля `person_id` в `favorite_food`, должно содержаться в таблцие `person`, т.е. в `person` должна быть запись содержащая нужное значение `person_id`.
