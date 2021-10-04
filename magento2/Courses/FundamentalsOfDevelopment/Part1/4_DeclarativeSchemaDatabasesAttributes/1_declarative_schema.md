# Декларативная схема

Декларативная схема в M2 содержит 4 элемента:

1. `etc/db_schema.xml` — используется для создания новых таблиц и модификации существующих
2. `etc/db_schema_whitlist.json` — этот файл включает модификацию другими модулями схемы, созданой в файле `etc/db_schema.xml` модуля
3. Дата патчи — позволяют производить операции над данными во время инсталяции модуля, например можно добавить новые продуктовые атрибуты.
4. Схема патчи — позволяют модифицировать схему БД с помощью SQL операций. Обычно достаточно модифицироать схему с помощью `etc/db_schema.xml`, однако иногда требуется спцифичные функции которые там не поддерживаются.

Изменения в схеме/данных будут применены при запуске команды `bin/magento setup:upgrade`.

## db_schema.xml

```xml
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
    <table name="wishlist" resource="default" engine="innodb" comment="Wishlist main Table">
        <column xsi:type="int" name="wishlist_id" padding="10" unsigned="true" nullable="false" identity="true"
                comment="Wishlist ID"/>
        <column xsi:type="int" name="customer_id" padding="10" unsigned="true" nullable="false" identity="false"
                default="0" comment="Customer ID"/>
        <column xsi:type="smallint" name="shared" padding="5" unsigned="true" nullable="false" identity="false"
                default="0" comment="Sharing flag (0 or 1)"/>
        <column xsi:type="varchar" name="sharing_code" nullable="true" length="32" comment="Sharing encrypted code"/>
        <column xsi:type="timestamp" name="updated_at" on_update="false" nullable="true" comment="Last updated date"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="wishlist_id"/>
        </constraint>
        <constraint xsi:type="foreign" referenceId="WISHLIST_CUSTOMER_ID_CUSTOMER_ENTITY_ENTITY_ID" table="wishlist"
                    column="customer_id" referenceTable="customer_entity" referenceColumn="entity_id"
                    onDelete="CASCADE"/>
        <constraint xsi:type="unique" referenceId="WISHLIST_CUSTOMER_ID">
            <column name="customer_id"/>
        </constraint>
        <index referenceId="WISHLIST_SHARED" indexType="btree">
            <column name="shared"/>
        </index>
    </table>
</schema>
```

`db_schema.xml` мерджатся, как и остальные конфигурационные файлы.

`<table>` — нода таблицы, используется для создания/модификации таблицы, может иметь атрибуты `name`, `resource`, `engine`, `comment`. Внутри `<table>` могут содержаться:

* `<column>` отвечает за колоники таблицы, имеет атрибуты:
  * xsi:type — тип колоники, допустивые:
    * blob
    * boolean
    * date
    * datetime
    * int
    * smallint
    * bigint
    * tinyint
    * decimal
    * float
    * double
    * real
    * text
    * timestamp
    * varbinary
    * varchar
  * default — дефолтное значение
  * disabled — отключает/удаляет опрделённую таблицу, колонку, ограничение или индекс
  * identify — флаг о том, что колонка это primary key
  * nullable — флаг для обозначения может ли колона быть NULL
  * length ­— длина для varchar
  * padding — размер для int 
  * scale|percision — количество числел дробной части
  * unsigned — флаг беззнаковости числа
* primary key содержится в `<constraint xsi:type="primary" referenceId="PRIMARY">`
* foreign key содержится в  `<constraint xsi:type="foreign"`, атрибуты:
  * referenceId — имя ключа
  * table — таблица где содержится ключ
  * column — колонка с ключём
  * referenceTable — таблица, на которую ссылается ключ
  * referenceColumn — колонка, на которую ссылается ключ
  * onDelete — определяет действие при удалении записи
* Уникальный ключ содержится в  `constraint xsi:type="unique"`, referenceId — имя ключа. Внутри себя содержит ноды `<column name="">`  с колонками входящими в ключ.
* индекс содержится в  `<index>`, внутри себя содержит ноды `<column name="">`  с колонками входящими в индекс. Атрибуты:
  * referenceId — имя индекса
  * indexType — тип индекса

## db_schema_whitlist.json

Пр умолчанию маджента не позволяет другим модулям модифицировать db_schema.xml.

Пр необходимости можно разрешить операции над схемой через `db_schema_whitlist.json`, где элементы из `db_schema.xml` перечисляются в json формате и указывается какие можно модифицировать в других модулях.

```json
{
    "wishlist": {
        "column": {
            "wishlist_id": true,
            "customer_id": true,
            "shared": true,
            "sharing_code": true,
            "updated_at": true
        },
        "index": {
            "WISHLIST_SHARED": true
        },
        "constraint": {
            "PRIMARY": true,
            "WISHLIST_CUSTOMER_ID_CUSTOMER_ENTITY_ENTITY_ID": true,
            "WISHLIST_CUSTOMER_ID": true
        }
    }
}
```

`db_schema_whitlist.json` нет необходимости создавать вручную, они генерируются командой:
```bash
bin/magento setup:db-declaration:generate-whitelist --module-name=<Custom_Module>
```

## Дата патч

Дата патичи находятся в директории _<module_die>/Setup/Path/Data_ они должны реализовывать [`Magento\Framework\Setup\Patch\DataPatchInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Setup/Patch/DataPatchInterface.php), также они могут реализоывавать [`Magento\Framework\Setup\Patch\PatchVersionInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Setup/Patch/PatchVersionInterface.php) используемый для инсталяции в M2.0-2.2. Для новых версий M2 указание номера версии не обязательно, вместо них используются зависимости и алиасы.

Ещё один важный элемент пачтей, это инстанс объекта [`Magento\Framework\Setup\ModuleDataSetupInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Setup/ModuleDataSetupInterface.php) он предоставляет доступ к основным вещам необходимым для SQL манипуляций, напрмер `getConnection()` позволяющий запускать SQL или `getTable()` возвращающим имя таблицы по элиасу.

### Зависимости и элиасы

`getDependencies()` возвращет список патчей, от который зависит текущий патч, как правило содержит полные имена классов патчей.

```php
public static function getDependencies()
{
    return [
        SetNewResourceModelsPaths::class,
    ];
}
```

`getAliases()` возвращает список элиасов для патча, которые можно использовать в `getDependencies()` других патчей, обычно возвращет пустой массив, а для идентификации патчей используются классы. 

```php
public function getAliases()
{
    return [];
}
```

## Схема патч

Схема патичи находятся в директории _<module_die>/Setup/Path/Schema_ они должны реализовывать [`Magento\Framework\Setup\Patch\SchemaPatchInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Setup/Patch/SchemaPatchInterface.php), также они могут реализоывавать [`Magento\Framework\Setup\Patch\PatchVersionInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Setup/Patch/PatchVersionInterface.php) используемый для инсталяции в M2.0-2.2. Для новых версий M2 указание номера версии не обязательно, вместо них используются зависимости и алиасы.

Так-же, как и дата патчи, получает в конструкторе инстанс объекта [`Magento\Framework\Setup\ModuleDataSetupInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Setup/ModuleDataSetupInterface.php) в нём так-же есть методы для начала и конца транзации инсталяции патча: `startSetup()` и `endSetup()`.

## Команды миграции

Для миграции на декларативную схему с M2.0-2.2 есть специальные команды, которые конвертируют инсталяционные и обновляющие скрипты в `etc/db_schema.xml`:
```bash
bin/magento setup:install --convert-old-scripts=1
bin/magento setup:upgrade --convert-old-scripts=1
```
