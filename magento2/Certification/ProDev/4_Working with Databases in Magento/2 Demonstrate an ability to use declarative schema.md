# Demonstrate an ability to use declarative schema

>Demonstrate use of schema. How to manipulate columns and keys using declarative schema? What is the purpose of whitelisting? How to use Data and Schema patches? How to manage dependencies between patch files?

## Install и Upgrade скрипты

[Setup scripts in Magento 2](https://inchoo.net/magento-2/setup-scripts-magento-2/)

В M2, до 2.3 миграции БД происходят через PHP-скрипты, в отличии от M1, все изменения пишутся одни файлы:

* InstallData реализует [`InstallDataInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Setup/InstallDataInterface.php)
* UpgradeData реализует [`UpgradeDataInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Setup/UpgradeDataInterface.php)
* InstallSchema реализует [`InstallSchemaInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Setup/InstallSchemaInterface.php)
* UpgradeSchema реализует [`UpgradeSchemaInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Setup/UpgradeSchemaInterface.php)

Скрипты лежат в директории _`<module-dir>`/Setup_.

Для upgrade-скриптов используется такое разделение по версиям:
```php
if (version_compare($context->getVersion(), '1.0.3') < 0) {
    //code to upgrade to 1.0.3
}
```

Код для работы с БД схож с M1.

Запус миграции происходит командой:
```bash
php bin/magento setup:upgrade
```

## Декларативная схема

[Declarative Schema Overview](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/)

В 2.3 появился новый способ миграции — декларативная схема.

### Миграция на декларативную схему

[Migrate install/upgrade scripts to declarative schema](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/migration-commands.html)

### Конфигурация

[Configure declarative schema](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/db-schema.html)

В декларативной схеме структура БД описывается в _`<module-dir>`/etc/db\_schema.xml_ файлах.

Пример описание таблицы `catalog_product_entity_datetime` выглядит так:
```xml
<table name="catalog_product_entity_datetime" resource="default" engine="innodb"
           comment="Catalog Product Datetime Attribute Backend Table">
    <column xsi:type="int" name="value_id" padding="11" unsigned="false" nullable="false" identity="true" comment="Value ID"/>
    <column xsi:type="smallint" name="attribute_id" padding="5" unsigned="true" nullable="false" identity="false" default="0" comment="Attribute ID"/>
    <column xsi:type="smallint" name="store_id" padding="5" unsigned="true" nullable="false" identity="false" default="0" comment="Store ID"/>
    <column xsi:type="int" name="entity_id" padding="10" unsigned="true" nullable="false" identity="false" default="0" comment="Entity ID"/>
    <column xsi:type="datetime" name="value" on_update="false" nullable="true" comment="Value"/>
    <constraint xsi:type="primary" referenceId="PRIMARY">
        <column name="value_id"/>
    </constraint>
    <constraint xsi:type="foreign" referenceId="CAT_PRD_ENTT_DTIME_ATTR_ID_EAV_ATTR_ATTR_ID" table="catalog_product_entity_datetime" column="attribute_id" referenceTable="eav_attribute" referenceColumn="attribute_id" onDelete="CASCADE"/>
    <constraint xsi:type="foreign" referenceId="CAT_PRD_ENTT_DTIME_ENTT_ID_CAT_PRD_ENTT_ENTT_ID" table="catalog_product_entity_datetime" column="entity_id" referenceTable="catalog_product_entity" referenceColumn="entity_id" onDelete="CASCADE"/>
    <constraint xsi:type="foreign" referenceId="CATALOG_PRODUCT_ENTITY_DATETIME_STORE_ID_STORE_STORE_ID" table="catalog_product_entity_datetime" column="store_id" referenceTable="store" referenceColumn="store_id" onDelete="CASCADE"/>
    <constraint xsi:type="unique" referenceId="CATALOG_PRODUCT_ENTITY_DATETIME_ENTITY_ID_ATTRIBUTE_ID_STORE_ID">
        <column name="entity_id"/>
        <column name="attribute_id"/>
        <column name="store_id"/>
    </constraint>
    <index referenceId="CATALOG_PRODUCT_ENTITY_DATETIME_ATTRIBUTE_ID" indexType="btree">
        <column name="attribute_id"/>
    </index>
    <index referenceId="CATALOG_PRODUCT_ENTITY_DATETIME_STORE_ID" indexType="btree">
        <column name="store_id"/>
    </index>
</table>
```

#### Структура `db_schema.xml`

* `schema` — корневая нода файла:
```xml
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
```
* `table` — каждый `db_schema.xml` должен содержать одну или более нод `table`, они представляют из себя описание таблицы БД. Атрибуты:
  * `name` — имя таблицы.
  * `engine` — SQL движок, может быть `innodb` или `memory`.
  * `resource` — [шард](https://en.wikipedia.org/wiki/Shard_(database_architecture)) БД в который устанавливается таблица, может быть `default`, `checkout`, `sales`.
  * `comment` — комментарий.
`table` может содержать такие подноды:
  * `column` — описание колонки
  * `constraint` — описание ограничений
  * `index` — индексы

При создании новых таблиц нужно [генерировать `db_schema_whitelist.json`](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/migration-commands.html#create-whitelist) это сделано для обратной совеместмости.

комадна:
```bash
bin/magento setup:db-declaration:generate-whitelist
```

#### Переименование таблиц

Декларативная схема позволяет переименовывать таблицы с переносом данных из старой в новую, для этого в таблице нужно указать `onCreate="migrateDataFromAnotherTable(old_table_name)"`:
```diff
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
+    <table name="new_declarative_table" onCreate="migrateDataFromAnotherTable(declarative_table)">
-    <table name="declarative_table">
        <column xsi:type="int" name="id_column" padding="10" unsigned="true" nullable="false" comment="Entity Id"/>
        <column xsi:type="int" name="severity" padding="10" unsigned="true" nullable="false" comment="Severity code"/>
        <column xsi:type="varchar" name="title" nullable="false" length="255" comment="Title"/>
        <column xsi:type="timestamp" name="time_occurred" padding="10" comment="Time of event"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="id_column"/>
        </constraint>
    </table>
</schema>
```

#### Добавление/Удаление колонок

Для добавление/удаления достаточно просто изменить определения таблицы в `db_schema.xml` и перегенерировать `db_schema_whitelist.json`.

#### Переименование колонок

Меняется в `db_schema.xml`, для переноса данных указывается `onCreate="migrateDataFrom(old_column_name)"`

#### Индексы и ключи

Добавление индекса:
```diff
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
    <table name="declarative_table">
        <column xsi:type="int" name="id_column" padding="10" unsigned="true" nullable="false" comment="Entity Id"/>
        <column xsi:type="int" name="severity" padding="10" unsigned="true" nullable="false" comment="Severity code"/>
        <column xsi:type="text" name="title" nullable="false" length="255" comment="Title"/>
        <column xsi:type="timestamp" name="time_occurred" padding="10" comment="Time of event"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="id_column"/>
        </constraint>
+       <index referenceId="INDEX_SEVERITY" indexType="btree">
+           <column name="severity"/>
+       </index>
    </table>
</schema>
```
Добавление внешнего ключа
```diff
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
    <table name="declarative_table">
        <column xsi:type="int" name="id_column" padding="10" unsigned="true" nullable="false" comment="Entity Id"/>
        <column xsi:type="int" name="severity" padding="10" unsigned="true" nullable="false" comment="Severity code"/>
        <column xsi:type="varchar" name="title" nullable="false" length="255" comment="Title"/>
        <column xsi:type="timestamp" name="time_occurred" padding="10" comment="Time of event"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="id_column"/>
        </constraint>
+       <constraint xsi:type="foreign" referenceId="FL_ALLOWED_SEVERITIES" table="declarative_table"
+               column="severity" referenceTable="severities" referenceColumn="severity_identifier"
+               onDelete="CASCADE"/>
    </table>
</schema>
```
>Внешние ключ. могут быть созданы только для таблиц которые созданы через декларативную схему.

### Патчи для данных и схема

[Develop data and schema patches](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/data-patches.html)
[Create a product attribute data patch with Magento 2.3's declarative schema](https://markshust.com/2019/02/19/create-product-attribute-data-patch-magento-2.3-declarative-schema/)

Патичи используеются для обновления данных и схемы БД. Патчи запускаются один раз при обновлении и заносятся в таблицу `patch_list`.

Патчи хранятся в 
* Для обновления схемы в _`<module-dir>`/Setup/Patch/Schema_ и реализуют [`Magento\Framework\Setup\Patch\SchemaPatchInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Setup/Patch/SchemaPatchInterface.php)
* Для обновления данных в _`<module-dir>`/Setup/Patch/Data_ и реализуют [`Magento\Framework\Setup\Patch\DataPatchInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Setup/Patch/DataPatchInterface.php)

Для отката данных (при удалении модуля) патч должен реализовать [`Magento\Framework\Setup\Patch\PatchRevertableInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Setup/Patch/PatchRevertableInterface.php)

Патчи выполяются при команде:
```bash
php bin/magento setup:upgrade
```

В патче можно указать зависимости (другие патчи), необходимые для его работы:
```php
public static function getDependencies()
{
    return [
        \SomeVendor\SomeModule\Setup\Patch\Data\SomePatch::class
    ];
}
```

### Версии

Декларативна схена заменяет `setup_module` версию из модуля на версию из композера.
