# Системные конфигурации

Важные особености системных конфигов:

* Системные конфиги нацелены на пользователей, а не разработчиков, поэтому туда не включаются технические аспекты сложные для понимания пользователя. Например, там нет порядка подсчёта итогов в корзине, но есть порядок вывода итогов.
* Все значения конфигурации хранятся в БД, в таблице `core_config_data`.
* Опции конфигурации могут иметь скопы действия: global, website, store.
* Все конфигурации настраиваются через конфигурационные файлы: `system.xml`.
* Путь конфигов состоит из 3х(или более, в зависимости от вложенности групп) частей, разделённых слешем `/`, это: секция, группа (или группы разделённые `/`), поле. Например конфиг "Locale" находится по пути: `general/locale/code`.

## system.xml

Кофиги настраиваются в файлах `system.xml` модулей, в __etc/adminhtml__. Файлы валидируются через файл [`system_file.xsd`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Config/etc/system_file.xsd).

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <tab id="catalog" translate="label" sortOrder="200">
            <label>Catalog</label>
        </tab>
        <tab id="advanced" translate="label" sortOrder="999999">
            <label>Advanced</label>
        </tab>
        <section id="catalog" translate="label" type="text" sortOrder="40" showInDefault="1" showInWebsite="1" showInStore="1">
            <class>separator-top</class>
            <label>Catalog</label>
            <tab>catalog</tab>
            <resource>Magento_Catalog::config_catalog</resource>
            <group id="fields_masks" translate="label" type="text" sortOrder="90" showInDefault="1" showInWebsite="1" showInStore="1">
                <label>Product Fields Auto-Generation</label>
                <field id="sku" translate="label comment" type="text" sortOrder="10" showInDefault="1" showInWebsite="0" showInStore="0" canRestore="1">
                    <label>Mask for SKU</label>
                    <comment>Use {{name}} as Product Name placeholder</comment>
                </field>
                <field id="meta_title" translate="label comment" type="text" sortOrder="20" showInDefault="1" showInWebsite="0" showInStore="0" canRestore="1">
                    <label>Mask for Meta Title</label>
                    <comment>Use {{name}} as Product Name placeholder</comment>
                </field>
                <field id="meta_keyword" translate="label comment" type="text" sortOrder="30" showInDefault="1" showInWebsite="0" showInStore="0" canRestore="1">
                    <label>Mask for Meta Keywords</label>
                    <comment>Use {{name}} as Product Name or {{sku}} as Product SKU placeholders</comment>
                </field>
                <field id="synchronize_with_backend" translate="label" type="select" showInDefault="1" canRestore="1">
                    <label>Synchronize widget products with backend storage</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="date_fields_order" translate="label" type="select" sortOrder="2" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
                    <label>Date Fields Order</label>
                    <frontend_model>Magento\Catalog\Block\Adminhtml\Form\Renderer\Config\DateFieldsOrder</frontend_model>
                </field>
            </group>
        </section>
    <system>
</config>
```

Внутри файла конфига содержатся два основных раздела:

* tab — содержит определения вкладок конфигурации
* section — содержит описания опций конфигурации, здесь определяется такая информация как:
  * вкладку раздела  —`tab`
  * ACL ресурс — `resource`
  * Имя раздела — `label`
  * CCS класс — `class`

Далее в `section` содержится раздел `group`, которые определяет группы конфигов, содержащиеся в секциях, которые содержатся во вкладках. `group` содержит своё название(`label`) и раздел `field`, который содержит поля конфигурации, также может содержать вложенные `group`. 

Для идентификации у нод `tab`, `section`, `group` и `field` есть атрибут `id`.

### Field

Аттрибуты `field`:

* `showInDefault`, `showInWebsite`, `showInStore` — указывают скопы действия и видимости конфига, напрмер "`showInDefault=1`, `showInWebsite=0`, `showInStore=0`" указывает на глобальный скоп.
* `id` — идентификатор
* `translate` — указывает какие дочерние ноды нужно перевести
* `type` — тип опции, доступные значения:
    * text
    * select
    * image
    * textarea
    * multiselect
* `sortOrder` — значения порядка сортировки
* `canRestore` — показывает флажок рядом со значениям "использовать системное значение", позволяющий использовать системное значение, вместо заданного

В filed могут содержатся такие элементы:

* `label` — имя опции конфига
* `comment` — комментарий
* `tooltip` — сообщение, которое отображется при наведении курсова на опцию
* `frontend_class` — CSS класс
* `frontend_model` — используется для кастомизации вывода опции в админке
* `backend_model` — класс вызываемый при сохранении опции, может содежрать бекендовые валдиации или иные действия выполняемые при сохранении опции
* `source_model` — используется для предоставления списка значений для опции типа select
* `config_path` — определяет альтернативный путь для опции, используемый вместо стандартного пути
* `validate` — CSS-класс используемый для валидации на фронте
* `can_by_empty` — флаг определяющий может ли опция не иметь значения (используется для типа `multiselect`)
* `if_module_enabled` — условия определяющие опцию только при включении определённых модулей
* `base_url` — используется для опций типа `image`
* `hide_in_single_store_mode` — определяет, должна ли опция быть доступна в режими одного стора, напимер `price_scope` не имеет смысла в таком режиме.
* `uploader_dir` — используется для опций типа `image`
* `depends` — определяет от какиих опций зависит эта опция (если условия не соблюдаются, то эта опция скрывается). [Пример](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Paypal/etc/adminhtml/system/express_checkout.xml#L59)
* `attribute` — позволяет определить кастомные поля опции. [Пример](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Paypal/etc/adminhtml/system/express_checkout.xml#L14)
* `requires` — делает опцию доступной только при активации другой `field`/`group`. [Пример](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Paypal/etc/adminhtml/system/express_checkout.xml#L136)

#### Source model

Пример конфигурации `<source_model>`:
```xml
<field id="list_mode" translate="label" type="select" sortOrder="1" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
    <label>List Mode</label>
    <source_model>Magento\Catalog\Model\Config\Source\ListMode</source_model>
</field>
```

Класс [`Magento\Catalog\Model\Config\Source\ListMode`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Catalog/Model/Config/Source/ListMode.php) реализует интерфейс [`Magento\Framework\Option\ArrayInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Option/ArrayInterface.php) и реализует его метод `toOptionArray()`:
```php
public function toOptionArray()
{
    return [
        ['value' => 'grid', 'label' => __('Grid Only')],
        ['value' => 'list', 'label' => __('List Only')],
        ['value' => 'grid-list', 'label' => __('Grid (default) / List')],
        ['value' => 'list-grid', 'label' => __('List (default) / Grid')]
    ];
}
```

Следует обратить внимание, что интрефейс [`Magento\Framework\Option\ArrayInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Option/ArrayInterface.php) признан устаревшим, вместо него стоит использовать [`Magento\Framework\Data\OptionSourceInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Data/OptionSourceInterface.php)

Прмечание: `__()` — функция локализации строки, в M2 она глобальная.

#### Frontend model

Пример конфигурации `<frontend_model>`:
```xml
<field id="date_fields_order" translate="label" type="select" sortOrder="2" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
    <label>Date Fields Order</label>
    <frontend_model>Magento\Catalog\Block\Adminhtml\Form\Renderer\Config\DateFieldsOrder</frontend_model>
</field>
```

Класс [`Magento\Catalog\Block\Adminhtml\Form\Renderer\Config\DateFieldsOrder`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Catalog/Block/Adminhtml/Form/Renderer/Config/DateFieldsOrder.php) наследует [`Magento\Config\Block\System\Config\Form\Field`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Config/Block/System/Config/Form/Field.php) который реализует интерфейс [`Magento\Framework\Data\Form\Element\Renderer\RendererInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Data/Form/Element/Renderer/RendererInterface.php) и наследуется от [`Magento\Backend\Block\Template`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Backend/Block/Template.php). В самом классе `DateFieldsOrder` переопределяется метод `_getElementHtml()` которые генерирует HTML для значения опции:
```php
protected function _getElementHtml(AbstractElement $element)
{
    $_options = ['d' => __('Day'), 'm' => __('Month'), 'y' => __('Year')];

    $element->setValues($_options)->setClass('select-date')->setName($element->getName() . '[]');
    if ($element->getValue()) {
        $values = explode(',', $element->getValue());
    } else {
        $values = [];
    }

    $_parts = [];
    $_parts[] = $element->setValue(isset($values[0]) ? $values[0] : null)->getElementHtml();
    $_parts[] = $element->setValue(isset($values[1]) ? $values[1] : null)->getElementHtml();
    $_parts[] = $element->setValue(isset($values[2]) ? $values[2] : null)->getElementHtml();

    return implode(' <span>/</span> ', $_parts);
}
```

# Меню

Меню конфигурируется в `menu.xml` находящихся в модулях, в _etc/adminhtml_:

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Backend:etc/menu.xsd">
    <menu>
        <add id="Magento_Catalog::catalog" title="Catalog" translate="title" module="Magento_Catalog" sortOrder="20" dependsOnModule="Magento_Catalog" resource="Magento_Catalog::catalog"/>
        <add id="Magento_Catalog::catalog_products" title="Products" translate="title" module="Magento_Catalog" sortOrder="10" parent="Magento_Catalog::inventory" action="catalog/product/" resource="Magento_Catalog::products"/>
    </menu>
</config>
```

# ACL

* ACL — это механизм ограничения доступа к определённым ресурсам в админке
* Доступ на любую страницу определяется как ALC-ресурс
* ACL проверки реализуют метод `_isAllowed()`

## _isAllowed()

Проверка ACL происходит в методах [`_isAllowed()`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Backend/App/AbstractAction.php#L106) контроллеров админки, в [`Magento\Backend\App\AbstractAction`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Backend/App/AbstractAction.php) содержится такая реализация:
```php
protected function _isAllowed()
{
    return $this->_authorization->isAllowed(static::ADMIN_RESOURCE);
}
```
Т.е. в большинстве случаев в дочерних котроллерах достаточно переопределить константу `ADMIN_RESOURCE`, указав в ней ид ресурса контроллера, например `Mage_Catalog::categories`.

## Конфигурация

ACL конфигурируется в файлах `acl.xml`, находящихся в модулях, в _etc_:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Acl/etc/acl.xsd">
    <acl>
        <resources>
            <resource id="Magento_Backend::admin">
                <resource id="Magento_Catalog::catalog" title="Catalog" translate="title" sortOrder="30">
                    <resource id="Magento_Catalog::catalog_inventory" title="Inventory" translate="title" sortOrder="10">
                        <resource id="Magento_Catalog::products" title="Products" translate="title" sortOrder="10">
                            <resource id="Magento_Catalog::edit_product_design" title="Edit Product Design" translate="title" />
                        </resource>
                        <resource id="Magento_Catalog::categories" title="Categories" translate="title" sortOrder="20">
                            <resource id="Magento_Catalog::edit_category_design" title="Edit Category Design" translate="title" />
                        </resource>
                    </resource>
                </resource>
            </resource>
        </resources>
    </acl>
</config>
```
