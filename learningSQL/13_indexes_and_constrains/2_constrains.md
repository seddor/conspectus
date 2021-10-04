# Ограничения

Ограничения — ограничивающие условия налагаемы на один или более столбцов таблицы.

Типы ограничений:
* Ограничения первичного ключа (primary-key constrains)
Идентифицирует столбец или столбцы, гарантирующие уникаольность в рамках таблицы
* Ограничения внешего ключа (Foreign-key constraints)
Накладывает ограничение, что столбец или столбцы могут содержать только значения, которые являются первичными для другой таблицы, с которой установлена свзь в ключе
* Ограничения уникальности (Unique constraints)
* Проврочные органичения целостности (Check constraints)
Ограничивает допустимые значения столбцов

>Если при работе с сервером MySQL требуются ограничения внешнего ключа, в таблицах должен использоваться механизм хранения InnoDB.

## Создание

Обычно ограничения указываются при созаднии таблицы:
```SQL
CREATE TABLE product
    (product_cd VARCHAR(10) NOT NULL,
    name VARCHAR(50) NOT NULL,
    product_type_cd VARCHAR (10) NOT NULL,
    date_offered DATE,
    date_retired DATE,

    CONSTRAINT fk_product_type_cd FOREIGN KEY (product_type_cd)
    REFERENCES product_type (product_type_cd),
    CONSTRAINT pk_product PRIMARY KEY (product_cd)
);
```
Здесь накладываются ограничения первичного ключа на `product_cd` и ограничения внешнего ключа на `product_type_cd`.

Так же это можно сделать отдельно:
```SQL
ALTER TABLE product
    ADD CONSTRAINT pk_product PRIMARY KEY (product_cd);

ALTER TABLE product
    ADD CONSTRAINT fk_product_type_cd FOREIGN KEY (product_type_cd)
    REFERENCES product_type (product_type_cd);
```

## Удаление

```SQL
ALTER TABLE product
    DROP PRIMARY KEY;

ALTER TABLE product
    DROP FOREIGN KEY fk_product_type_cd;
```

## Ограничения и индексы

Иногда создание ограничения ведёт за собой автоматическое созадние индекса. Такое поведение зависит от конкретной СУБД.

В MySQL так:
* Ограничения первичного ключа — формирует уникальный индекс
* Ограничение внешнего ключа — формируект индекс
* Ограничения уникальности — формирует уникальный индекс.

## Каскадные ограничения

Для внешних ключей можно установить какие действия будут производится со связаными данными при измении или удалени их в связанной таблице:
```SQL
ALTER TABLE product
    ADD CONSTRAINT fk_product_type_cd FOREIGN KEY (product_type_cd)
    REFERENCES product_type (product_type_cd)
    ON UPDATE CASCADE;
```
Теперь при изменении записи из product_type будут так-же изменены значения в связанном столбце `product_type_cd`.
Для действий при удалении:
```SQL
ALTER TABLE product
    ADD CONSTRAINT fk_product_type_cd FOREIGN KEY (product_type_cd)
    REFERENCES product_type (product_type_cd)
    ON DELETE CASCADE;
```

Возможные значения:
* `CASCADE` — каскадное измение/удаление связанных данных
* `SET NULL` — измения значения на NULL, в столбцы должен быть допустим NULL
* `RESTRICT` — запретиить измение/удаление, испоьзуется по умолчанию
* `NO ACTION` — то-же самое, что `RESTRICT`
* `SET DEFAULT` — изменяет значения связанных столбцов на дефолтные.
