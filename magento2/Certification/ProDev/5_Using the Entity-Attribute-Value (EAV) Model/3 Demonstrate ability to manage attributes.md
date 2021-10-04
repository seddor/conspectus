# Demonstrate ability to manage attributes

>Describe EAV attributes, including the frontend/source/backend structure. How would you add dropdown/multiselect attributes? What other possibilities do you have when adding an attribute (to a product, for example)?

>Describe how to implement the interface for attribute frontend models. What is the purpose of this interface? How can you render your attribute value on the frontend?

>Identify the purpose and describe how to implement the interface for attribute source models. For a given dropdown/multiselect attribute, how can you specify and manipulate its list of options?

>Identify the purpose and describe how to implement the interface for attribute backend models. How (and why) would you create a backend model for an attribute?

>Describe how to create and customize attributes. How would you add a new attribute to the product, category, or customer entities? What is the difference between adding a new attribute and modifying an existing one?

[How to Manage EAV Attributes Including interface/source/backend Structure in Magento 2](https://belvg.com/blog/how-to-manage-eav-attributes-including-interface-source-backend-structure-in-magento-2.html)

Атрибуты добавляются через инсталяционные скрипты, или через патчи(для >= 2.3), подробнее об этом в 5.1, для продуктов атрибуты (для всего в EE) можно создавать из админки.

## Frontend model

Предназначена для рендеринга атрибута в админке, фронтэнд модели наследуются от [`Magento\Eav\Model\Entity\Attribute\Frontend\AbstractFrontend`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/Entity/Attribute/Frontend/AbstractFrontend.php) который реализует [`Magento\Eav\Model\Entity\Attribute\Frontend\FrontendInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/Entity/Attribute/Frontend/FrontendInterface.php)


Для измения вывода преопределяется метод `getValue()`

## Source model

Используются для предоставления возможных значений для `select`/`multiselect` атрибутов, наследуют класс [`Magento\Eav\Model\Entity\Attribute\Source\AbstractSource`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/Entity/Attribute/Source/AbstractSource.php) реализующий [`Magento\Eav\Model\Entity\Attribute\Source\SourceInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/Entity/Attribute/Source/SourceInterface.php)

Нужно реализовать метод `getAllOptions()`, формат опций такой-же как был в M1:
```php
class TestSource extends \Magento\Eav\Model\Entity\Attribute\Source\AbstractSource
{
   public function getAllOptions()
   {
       if (!$this->_options) {
           $this->_options = [
               ['label' => __('Label 1'), 'value' => 'value 1'],
               ['label' => __('Label 2'), 'value' => 'value 2'],
               ['label' => __('Label 3'), 'value' => 'value 3'],
               ['label' => __('Label 4'), 'value' => 'value 4']
           ];
       }
       return $this->_options;
   }
}
```

Для получения значений из значений атрибута (таблица `eav_attribute_option_value`) используется [`Magento\Eav\Model\Entity\Attribute\Source\Table`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/Entity/Attribute/Source/Table.php) она же используется по умолчанию если не задана source для атибутов с `frontend_input` равными multiselect или select.

## Backend model

Используется для модификации процеса загрузки/сохранения/удаление/валидации значений атрибута.

Бекэндовые модели наследуются от [`Magento\Eav\Model\Entity\Attribute\Backend\AbstractBackend`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/Entity/Attribute/Backend/AbstractBackend.php) реализующий [`Magento\Eav\Model\Entity\Attribute\Backend\BackendInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/Entity/Attribute/Backend/BackendInterface.php)

## Работа с атрибутами

Репозиторий атрибутов: [`Magento\Eav\Model\AttributeRepository`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/AttributeRepository.php) реализует [`Magento\Eav\Api\AttributeRepositoryInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Api/AttributeRepositoryInterface.php).

Для получения атрибута используется метод [`get()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/AttributeRepository.php#L161). В него нужно передать тип атрибута и код атрибута, в результате он вернёт инстанст [`Magento\Eav\Model\Entity\Attribute`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/Entity/Attribute.php) реализующий [`Magento\Eav\Api\Data\AttributeInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Api/Data/AttributeInterface.php).


### Созадание атрибута, группа/набор, измеение свойств

Для созадния/изменения атрибута используется класс [`Magento\Eav\Setup\EavSetup`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Setup/EavSetup.php). 

* [`addAttribute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Setup/EavSetup.php#L815) — используется для создание аттрибута, если в массиве сойств указать `group`, то атрибут добавится во всех наборы атрибутов.
* [`updateAttribute()`]() — используется для измение свойств атрибута.

## Sales атрибуты

Хотя фактически атрибуты для sales сущностей (заказы, инвойсы, кредит мемо и т.п.) сейчас не используются и все данные пишутся в плоские таблицы, однако для добавление полей/атрибутов к этим сущностям есть специальный инсталлер который следует использовать [`Magento\Sales\Setup\SalesSetup`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/Setup/SalesSetup.php)
