# Demonstrate ability to use EAV model concepts

>Describe the EAV hierarchy structure. What happens when a new attribute is added to the system? What is the role of attribute sets and attribute groups? How are attributes presented in the admin?

>Describe how EAV data storage works in Magento. Which additional options do you have when saving EAV entities? How do you create customizations based on changes to attribute values?

>Describe the key differences between EAV and flat table collections. In which situations would you use EAV for a new entity? What are the pros and cons of EAV architecture?

[Magento 2 EAV: обзор структур данных](https://habr.com/ru/post/441122/)

В M2, как и в M1, используется [EAV (Entity-Attribute-Value)](https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model). 

## Структура

EAV-таблицы в M2 можно разделить на 3 группы:

* `eav_attribute`
* `eav_entity`
* `eav_form`

### `eav_form`

Нужна для отображения некоторых EAV-данных в UI, непосредствено к самим данным не относится.

### `eav_attribute`

В `eav_attribute` хранятся все атрибуты и их базовые (общие) поля.

### `eav_entity`

В `eav_entity_type` хранятся сущности для которых предпологается хранение в EAV, в 2.4 это:

* `customer`
* `customer_address`
* `catalog_category`
* `catalog_product`
* `order`
* `invoice`
* `creditmemo`
* `shipment`

Для последних 4х по факту EAV ещё не используется (как и в M1).

`eav_entity_attribute` используется для упорядочивания атрибутов в группах.

Есть ещё несколько таблиц, которые относятся к eav, но в данный момент не исользуются:

* `eav_entity`
* `eav_entity_datetime`
* `eav_entity_decimal`
* `eav_entity_int`
* `eav_entity_store`
* `eav_entity_text`
* `eav_entity_varchar`

### Значения

Сами же значения хранятся в специальных таблицах, имя которых строится таким образом:
```
{тип сущности}_{тип данных}
```

Примеры типов данных:

* `datetime`
* `decimal`
* `int`
* `text`
* `varchar`

Для создание нового типа данных нужно создать соотвующую таблицу и указать её в `backend_table` атрибута с данным этого типа.

Примеры типов сущностей:

* `catalog_category_entity`
* `catalog_product_entity`
* `customer_address_entity`
* `customer_entity`
* `eav_entity`

### Тип значения атрибиута

Тип значения хранится в `eav_attribute.backend_type`, он влияет на то, где будет храниться значение:

* `static` — будет храниться в таблице самой сущности, например у кастомеров в таблице есть атрибут `email`. Колонки для таких атрибутов не добавляются в таблицу автоматически, поэтому их нужно добавить вручную. Статичные атрибуты можно использовать для быстрого доступа к данным, т.к. эти атрибуты не требуют затратной загрузки и сразу содержатся в таблице сущности, при этом стоит учитывать, что статичные атрибуты поддерживают данные только в глобальном скоупе (т.е. нельзя задать различные значения для разных сторов/сайтов).
* `datetime`, `decimal`, `int`, `text`, `varchar` — в отдельной таблице соответствующей сущности и соответствующего типа.

Структра таблицы со значениями такая же как в M1 и практически одинакова для всех стандартных типов:

* `value_id` — ИД значения
* `attribute_id` — ИД атрибута
* `store_id` — ИД стора
* `entity_id` — ИД сущности
* `value` — значения, тип колонки соответствует типу значения

Для таблицы действует ограничение на уникальности по сочитанию `entity_id`, `attribute_id`, `store_id`.

## Атрибуты для продуктов

Для продуктов есть ещё две EAV таблицы:

* `eav_attribute_set` — объеденяет атрибуты в наборы, которые присваиваются продукту при создании, тем самым ограничивая набор атрибутов продукта.
* `eav_attribute_group` — группы атрибутов на форме продукта в админке

## Создание аттрибутов

Атрибуты для продуктов можно создавать из админки или программно, для остальных сущностей только программным путём (В EE можно из админки для всех атрибутов).

[How to Add EAV Attribute in Magento 2](https://www.mageplaza.com/devdocs/how-create-eav-attribute-magento-2.html)
[Create a product attribute data patch with Magento 2.3's declarative schema](https://markshust.com/2019/02/19/create-product-attribute-data-patch-magento-2.3-declarative-schema/)

## Extension attributes

[EAV and extension attributes](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/attributes.html) и её [перевод](https://sfhub.ru/?p=448)
[An Introduction to Extension Attributes](https://store.fooman.co.nz/blog/an-introduction-to-extension-attributes.html)

Extension attributes позволяют добавлять дополнительные поле к моделям. Они конфигурируются в _`<module-dir>`/etc/extension\_attrubutes.xml_. Они могут быть как значением так и объектом.  

Пример конфигурации:
```xml
<extension_attributes for="Magento\Directory\Api\Data\CountryInformationInterface">
    <attribute code="currency_information" type="Chapter3\Database\Api\Data\CurrencyInformationInterface" />
</extension_attributes>
```
После этого маджента добавит методы `getCurrencyInformation()` и `setCurrencyInformation()` в сгенерированный файл интерфеса и класса для extension attributes данной модели.

Для того чтобы модель поддерживала extension attributes у класса модели должен быть метод `getExtensionAttributes()`. Для моделей можно использовать класс [`Magento\Framework\Api\AbstractExtensibleObject`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/AbstractExtensibleObject.php) и [`Magento\Framework\Model\AbstractExtensibleModel`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Model/AbstractExtensibleModel.php) (для EAV) или реализовать метод самостоятельно.

## Отличие custom и extension attributes

Custom attributes грузятся из таблиц EAV моделей. Extension attributes тоже могут грузится оттуда, но как правило они используют иные структуры для загрузки данных (другие таблицы в БД, кэш, ФС, стороние API).

## Дополнительно

[How EAV Data Storage Works in Magento 2](https://belvg.com/blog/how-eav-data-storage-works-in-magento-2.html)
