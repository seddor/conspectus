# Управление атрибутами

## Структура

поля атрибута:

* `attribute_id` — идентификатор
* `entity_type_id` — ИД типа сущности
* `attribute_code` — человеко-читаемый, уникальный в приделах типа сущности код атрибута
* `attribute_model` — опциальная альтернативная модель для атрибута, по дефолту используется [`Magento\Eav\Model\Entity\Attribute`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Eav/Model/Entity/Attribute.php)
* `backend_model` — модель используемая для процессинга значений атрибута перед и после CRUD операций
* `backend_type` — тип атрибута, дефолтные (можно добавить свои):
    * static
    * varchar
    * datetime
    * int
    * text
    * decimal
* `backend_table` — опциально, таблица значений атрибута, если не указана будет использована таблица на основе `backend_type`.  
* `frontend_model` — модель используемая для отображения значения атрибута
* `frontend_input` — тип используемый для ввода значения атрибута в админке
* `frontend_label` — Дефолтное название атрибута, используется в админке
* `frontend_class` — опционально, CSS класс, будет добавлен к инпуту в админке, обычно используется для фронтовой валидации в админке
* `source_model` — модель используемая для предоставления набора значений атрибутов с типом вывода `select` и `multiselect`
* `is_required` — указывает обязателен ли атрибут, проверка происходит на уровен JS валидации.
* `default_value` — опционально, дефолтное значение для атрибута, отображается фронтедовой моделью если атрибут не имеет значение

Каждый атрибут может иметь некоторые типа-специфичные поля, которые хранятся в дополнительных таблицах, например `catalog_eav_attribute`.

## Мета информация

Для получения мета информации импользуется класс [`Magento\Eav\Model\Config`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Eav/Model/Config.php), основные методы:

* [`getAttribute($entityType, $code)`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Eav/Model/Config.php#L532)
* [`getEntityType($code)`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Eav/Model/Config.php#L423)

## Setup

Для работы с атрибутами в инсталяционных скриптах нужно использовать класс [`Magento\Eav\Setup\EavSetup`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Eav/Setup/EavSetup.php), он позволяет добавлять/удлаять/изменять атрибуты.

## Атрибутные модели

Основные атрибутные модели

* Backend — позволяет добавлять и работать со значениями атрибутов во время сохранения, загрузки, удалени. Примеры можно найти в [`Magento\Eav\Model\Entity\Attribute\Backend`](https://github.com/magento/magento2/tree/2.3/app/code/Magento/Eav/Model/Entity/Attribute/Backend)
* Source — предоставляет возможные значения для атрибутов c `frontend_input`, равное `select` и `multiselect`. Примеры можно найти в [`Magento\Eav\Model\Entity\Attribute\Source`](https://github.com/magento/magento2/tree/2.3/app/code/Magento/Eav/Model/Entity/Attribute/Source)
* Frontend — рендерит значения атрибута. Примеры можно найти в [`Magento\Eav\Model\Entity\Attribute\Frontend`](https://github.com/magento/magento2/tree/2.3/app/code/Magento/Eav/Model/Entity/Attribute/Frontend)
