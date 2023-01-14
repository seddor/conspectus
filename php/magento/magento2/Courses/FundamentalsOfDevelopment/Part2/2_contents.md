# Контент

## Гриды

Гриды в M2 реализованы через Ui компоненты.

Фильтры гридов в M2 отделены от данных.

Гриды используеются для:

* Рендеринга списков записей отдельных сущностей
* Предоставление сортировки, фильтрации, поиска в списке
* Позволяет запускать определнённые события для групп записей (например: активация, деактивация, удаление)


### Копоненты гридов

* Listing — фильтры, основной компонент грида, содержит остальные копоненты
* Columns — колонки, вклачает в себя список копонентов-колонок, каждый копонент представляет из себя определённую колонку грида и её данные. Колонки рендярятся JS.
* Filters — фильтры, включает в себя копоненты-фильтры, доступно несколько типов фильтров:
  * select
  * input
  * range
  * search
  * data
* Paging — пагинация, копонент отвечающий за пагинацию.
* Massactions — копонент выводящий меню массовых действий грида

#### Listing UI копонент

* Аналог grid widget из M1
* Содержит группу дочерних элементов, реализующие отдельные возможности грида
* Работает с коллекциями через `DataProvider`

Конфигурация в `definition.xml`:
```xml
<listing sorting="true" class="Magento\Ui\Component\Listing" component="uiComponent">
    <argument name="data" xsi:type="array">
        <item name="template" xsi:type="string">templates/listing/default</item>
        <item name="save_parameters_in_session" xsi:type="string">1</item>
        <item name="client_root" xsi:type="string">mui/index/render</item>
    </argument>
</listing>
```

* Класс: [`Magento\Ui\Component\Listing\Columns\Listing`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Listing.php)
* Корневой темплейт: [`templates/listing/default.xhtml`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/view/base/ui_component/templates/listing/default.xhtml)

Пример: грид для CMS страниц [`Magento/Cms/view/adminhtml/ui_component/cms_page_listing.xml`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Cms/view/adminhtml/ui_component/cms_page_listing.xml):
```xml
<argument name="data" xsi:type="array">
    <item name="js_config" xsi:type="array">
        <item name="provider" xsi:type="string">cms_page_listing.cms_page_listing_data_source</item>
    </item>
</argument>
<settings>
    <buttons>
        <button name="add">
            <url path="*/*/new"/>
            <class>primary</class>
            <label translate="true">Add New Page</label>
        </button>
    </buttons>
    <spinner>cms_page_columns</spinner>
    <deps>
        <dep>cms_page_listing.cms_page_listing_data_source</dep>
    </deps>
</settings>
```

* `js_config`, `settings/deps` — определяют конфигурацию JS
* `settings/buttons`  — определяют список кнопок которые добавляются в грид

#### DataSource

`DataSource` — UI копонент, предоставляющий данные для других копонентов. Для работы `DataSource` ему нужен `DataProvider` который будет извлекать нужные данные, он реализует специальный интерфейс для предоставления данных в формате JSON в `DataSource`.

Обычно для каждого инстанса UI компонетов имплементируется свой `DataProvider`.

Конфигурация `DataSource` из `definition.xml`:
```xml
<dataSource class="Magento\Ui\Component\DataSource"/>
```

`DataProvider` наследует класс [`Magento\Ui\DataProvider\AbstractDataProvider`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/DataProvider/AbstractDataProvider.php), он делает:

* Оборачивает операции коллекций, такие как: сортировка, фльтрация
* Реализует метод `getData()`, который используется для извелечения данных в JS и возвращает массив:
```php
public function getData()
{
    return $this->getCollection()->toArray();
}
```
* Имеет абстрактный метод `getCollection()`, который реализуется в определённом `DataProvider`.

Пример `dataSource` для списка продуктов:
```xml
<dataSource name="product_listing_data_source" component="Magento_Ui/js/grid/provider">
    <settings>
        <storageConfig>
            <param name="dataScope" xsi:type="string">filters.store_id</param>
        </storageConfig>
        <updateUrl path="mui/index/render"/>
    </settings>
    <aclResource>Magento_Catalog::products</aclResource>
    <dataProvider class="Magento\Catalog\Ui\DataProvider\Product\ProductDataProvider" name="product_listing_data_source">
        <settings>
            <requestFieldName>id</requestFieldName>
            <primaryFieldName>entity_id</primaryFieldName>
        </settings>
    </dataProvider>
</dataSource>
```

* `DataProvider` — класс [`Magento\Catalog\Ui\DataProvider\Product\ProductDataProvider`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Catalog/Ui/DataProvider/Product/ProductDataProvider.php), имя `DataProvider` должно совпадать с именем `DataSource`.
* атрибут `component` — это JS класс, загружающий данные. Обычно для гридов используется стандартный `Magento_Ui/js/grid/provider`.
* `param name="dataScope"` — определяет скоп данных, которые будут использоваться внутри JS модуля
* `updateUrl` — URL для обновления, обычно используется стандартный `mui/index/render`

Имя `DataSource` обычно состоит из двух частей: `<instanse_name>_data_source`.

[`Magento\Catalog\Ui\DataProvider\Product\ProductDataProvider`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Catalog/Ui/DataProvider/Product/ProductDataProvider.php) реазлиует методы:

* [`getData()`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Catalog/Ui/DataProvider/Product/ProductDataProvider.php#L79)
* [`addField()`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Catalog/Ui/DataProvider/Product/ProductDataProvider.php#L105)
* [`addFilter()`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Catalog/Ui/DataProvider/Product/ProductDataProvider.php#L117)

#### Колонки грида

Columns UI компонент представляет из себя контейнер сожержащий набор копонентов колонок (каждая из которых тоже UI компонент). 

Для EAV сущностей Magento генерирует список колонок автоматически.

Columns UI компонент в `definition.xml`:
```xml
<columns class="Magento\Ui\Component\Listing\Columns" component="Magento_Ui/js/grid/listing">
    <settings>
        <componentType>columns</componentType>
        <storageConfig>
            <namespace>current</namespace>
            <provider>ns = ${ $.ns }, index = bookmarks</provider>
        </storageConfig>
        <childDefaults>
            <param name="storageConfig" xsi:type="array">
                <item name="provider" xsi:type="string">ns = ${ $.ns }, index = bookmarks</item>
                <item name="root" xsi:type="string">columns.${ $.index }</item>
                <item name="namespace" xsi:type="string">current.${ $.storageConfig.root }</item>
            </param>
        </childDefaults>
    </settings>
</columns>
<column class="Magento\Ui\Component\Listing\Columns\Column" component="Magento_Ui/js/grid/columns/column">
    <settings>
        <componentType>column</componentType>
        <dataType>text</dataType>
    </settings>
</column>
```

* Класс Columns UI: [`Magento\Ui\Component\Listing\Columns`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Listing/Columns.php)
* Класс Column UI: [`Magento\Ui\Component\Listing\Columns\Column`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Listing/Columns/Column.php)

Для рендеринга колонок используетюся JS модули из [Magento/Ui/view/base/web/js/grid/columns/*](https://github.com/magento/magento2/tree/2.3/app/code/Magento/Ui/view/base/web/js/grid/columns):

[`column.js`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/view/base/web/js/grid/columns/column.js) — базовый JS модуль для колонок.

Пример конфигурации колонки для CMS страницы (cms_page_listing.xml):
```xml
<column name="is_active" component="Magento_Ui/js/grid/columns/select">
    <settings>
        <options class="Magento\Cms\Model\Page\Source\IsActive"/>
        <filter>select</filter>
        <editor>
            <editorType>select</editorType>
        </editor>
        <dataType>select</dataType>
        <label translate="true">Status</label>
    </settings>
</column>
```
Эта колонка использует тип `select`, для получения опций используется класс [`Magento\Cms\Model\Page\Source\IsActive`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Cms/Model/Page/Source/IsActive.php), для рендеринга используется [`select.js`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/view/base/web/js/grid/columns/select.js).

Для того, чтобы не отображать колонку используется `setting/visable`:
```xml
<column name="is_active" component="Magento_Ui/js/grid/columns/select">
    <settings>
        <options class="Magento\Cms\Model\Page\Source\IsActive"/>
        <filter>select</filter>
        <editor>
            <editorType>select</editorType>
        </editor>
        <dataType>select</dataType>
        <label translate="true">Status</label>
        <visable>false<visable/>
    </settings>
</column>
```

#### Фильтры

Фильтры в отличии от M1 теперь отделены от колонок, состав фильтров и колонок может отличатся. Сами фильтры являются UI компонентами.

`filters` в `definition.xml`:

```xml
<filters class="Magento\Ui\Component\Filters" component="Magento_Ui/js/grid/filters/filters" displayArea="dataGridFilters">
    <argument name="data" xsi:type="array">
        <item name="observers" xsi:type="array">
            <item name="column" xsi:type="string">column</item>
        </item>
    </argument>
    <settings>
        <dataScope>filters</dataScope>
        <storageConfig>
            <namespace>current.filters</namespace>
            <provider>ns = ${ $.ns }, index = bookmarks</provider>
        </storageConfig>
    </settings>
</filters>
```

* Класс [`Magento\Ui\Component\Filters`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Filters.php)

Применения фильтров зависит от имплементации `DataProvider`.

Стандартный `DataProvider`([`Magento\Framework\View\Element\UiComponent\DataProvider\DataProvider`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/View/Element/UiComponent/DataProvider/DataProvider.php)) использует [`Magento\Framework\View\Element\UiComponent\DataProvider\FilterPool`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/View/Element/UiComponent/DataProvider/FilterPool.php), который содержит метод [`applyFilters()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/View/Element/UiComponent/DataProvider/FilterPool.php#L38) который применяет фильтры.  Конфигурация этого метода и других фильтров, может быть сделана в `di.xml` для конкретного грида.

#### Массовые действия

Используются для примения каких-либо действий к набору записей. Список действий опредялет в конфигурация `data/config/actions`.

`massaction` в `definition.xml`:

```xml
<massaction class="Magento\Ui\Component\MassAction" component="Magento_Ui/js/grid/massactions" displayArea="bottom"/>
```

* Класс [`Magento\Ui\Component\MassAction`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/MassAction.php)
* JS модуль [`massaction.js`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/view/base/web/js/grid/massactions.js)

Пример для CMS страниц:
```xml
<massaction name="listing_massaction">
    <action name="delete">
        <settings>
            <confirm>
                <message translate="true">Are you sure you want to delete selected items?</message>
                <title translate="true">Delete items</title>
            </confirm>
            <url path="cms/page/massDelete"/>
            <type>delete</type>
            <label translate="true">Delete</label>
        </settings>
    </action>
    <action name="disable">
        <settings>
            <url path="cms/page/massDisable"/>
            <type>disable</type>
            <label translate="true">Disable</label>
        </settings>
    </action>
    <action name="enable">
        <settings>
            <url path="cms/page/massEnable"/>
            <type>enable</type>
            <label translate="true">Enable</label>
        </settings>
    </action>
    <action name="edit">
        <settings>
            <callback>
                <target>editSelected</target>
                <provider>cms_page_listing.cms_page_listing.cms_page_columns_editor</provider>
            </callback>
            <type>edit</type>
            <label translate="true">Edit</label>
        </settings>
    </action>
</massaction>
```

#### Пагинация

Пагинация используется для постраничного разделения списков.

`paging` в `definition.xml`:

```xml
<paging class="Magento\Ui\Component\Paging" component="Magento_Ui/js/grid/paging/paging" displayArea="bottom">
    <settings>
        <storageConfig>
            <namespace>current.paging</namespace>
            <provider>ns = ${ $.ns }, index = bookmarks</provider>
        </storageConfig>
    </settings>
</paging>
```

* Класс [`Magento\Ui\Component\Paging`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Paging.php)
* JS модуль [`paging.js`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/view/base/web/js/grid/paging/paging.js)

Пример конфигурации для грида продуктов:
```xml
<listingToolbar name="listing_top">
    <paging name="listing_paging"/>
</listingToolbar>
```

### Индекс грида

В `Magento_Sales` модуле содержится набор специальных таблиц сделанных для гридов в админке. Это таблицы заполянются на лету при сохранении сущности. Эти таблицы представляют из себя набор агрегированных данных. Они нужны для решения проблем со сложными запросами.

Таблица для грида заказов: `sales_order_grid`, таблица так-же имеет собственную коллекцию для рендеринга грида.

Конфигурация `dataSource` для заказов:
```xml
<dataSource name="sales_order_grid_data_source" component="Magento_Ui/js/grid/provider">
    <settings>
        <updateUrl path="mui/index/render"/>
    </settings>
    <aclResource>Magento_Sales::sales_order</aclResource>
    <dataProvider class="Magento\Framework\View\Element\UiComponent\DataProvider\DataProvider" name="sales_order_grid_data_source">
        <settings>
            <requestFieldName>id</requestFieldName>
            <primaryFieldName>main_table.entity_id</primaryFieldName>
        </settings>
    </dataProvider>
</dataSource>
```

Конфгурация `di.xml`:
```xml
<type name="Magento\Framework\View\Element\UiComponent\DataProvider\CollectionFactory">
    <arguments>
        <argument name="collections" xsi:type="array">
            <item name="sales_order_grid_data_source" xsi:type="string">Magento\Sales\Model\ResourceModel\Order\Grid\Collection</item>
            <item name="sales_order_invoice_grid_data_source" xsi:type="string">Magento\Sales\Model\ResourceModel\Order\Invoice\Grid\Collection</item>
            <item name="sales_order_shipment_grid_data_source" xsi:type="string">Magento\Sales\Model\ResourceModel\Order\Shipment\Grid\Collection</item>
            <item name="sales_order_creditmemo_grid_data_source" xsi:type="string">Magento\Sales\Model\ResourceModel\Order\Creditmemo\Grid\Collection</item>
            <item name="sales_order_view_invoice_grid_data_source" xsi:type="string">Magento\Sales\Model\ResourceModel\Order\Invoice\Orders\Grid\Collection</item>
            <item name="sales_order_view_shipment_grid_data_source" xsi:type="string">Magento\Sales\Model\ResourceModel\Order\Shipment\Order\Grid\Collection</item>
            <item name="sales_order_view_creditmemo_grid_data_source" xsi:type="string">Magento\Sales\Model\ResourceModel\Order\Creditmemo\Order\Grid\Collection</item>
        </argument>
    </arguments>
</type>
```

## Формы

Формы в M2 — это фреймворк, который создаёт пользовательский интерфейс для управления сущностями.

Контейнер формы содержит список действий доступных для формы (например: сохранение, удаление).

PHP класс для форм [`Magento\Ui\Component\Form`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Form.php), он наследует [`Magento\Ui\Component\AbstractComponent`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/AbstractComponent.php).

`form` в `definition.xml`:
```xml
<form class="Magento\Ui\Component\Form">
    <argument name="data" xsi:type="array">
        <item name="js_config" xsi:type="array">
            <item name="component" xsi:type="string">Magento_Ui/js/form/form</item>
        </item>
        <item name="template" xsi:type="string">templates/form/default</item>
    </argument>
</form>
```
* Класс [`Magento\Ui\Component\Form`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Form.php)
* JS модуль [`form.js`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/view/base/web/js/form/form.js)

Пример конфигурации кнопок на форме для формы клиента ([`customer_form.xml`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Customer/view/base/ui_component/customer_form.xml)):
```xml
<buttons>
    <button name="save_and_continue" class="Magento\Customer\Block\Adminhtml\Edit\SaveAndContinueButton"/>
    <button name="save" class="Magento\Customer\Block\Adminhtml\Edit\SaveButton"/>
    <button name="reset" class="Magento\Customer\Block\Adminhtml\Edit\ResetButton"/>
    <button name="order" class="Magento\Customer\Block\Adminhtml\Edit\OrderButton"/>
    <button name="resetPassword" class="Magento\Customer\Block\Adminhtml\Edit\ResetPasswordButton"/>
    <button name="unlock" class="Magento\Customer\Block\Adminhtml\Edit\UnlockButton"/>
    <button name="invalidateToken" class="Magento\Customer\Block\Adminhtml\Edit\InvalidateTokenButton"/>
    <button name="delete" class="Magento\Customer\Block\Adminhtml\Edit\DeleteButton"/>
    <button name="back" class="Magento\Customer\Block\Adminhtml\Edit\BackButton"/>
</buttons>
```

Классы кнопок реализуют [`Magento\Framework\View\Element\UiComponent\Control\ButtonProviderInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/View/Element/UiComponent/Control/ButtonProviderInterface.php)

Пример конфигурации `DataSource`:
```xml
<dataSource name="customer_form_data_source">
    <argument name="data" xsi:type="array">
        <item name="js_config" xsi:type="array">
            <item name="component" xsi:type="string">Magento_Ui/js/form/provider</item>
        </item>
    </argument>
    <settings>
        <validateUrl path="customer/index/validate"/>
        <submitUrl path="customer/index/save"/>
    </settings>
    <dataProvider class="Magento\Customer\Model\Customer\DataProviderWithDefaultAddresses" name="customer_form_data_source">
        <settings>
            <requestFieldName>id</requestFieldName>
            <primaryFieldName>entity_id</primaryFieldName>
        </settings>
    </dataProvider>
</dataSource>
```

### Fieldset

Fieldset — это компонент-контейнер содержащий в себе группу копонентов полей.

`fieldset` в `definition.xml`:
```xml
<fieldset class="Magento\Ui\Component\Form\Fieldset">
    <argument name="data" xsi:type="array">
        <item name="js_config" xsi:type="array">
            <item name="component" xsi:type="string">Magento_Ui/js/form/components/fieldset</item>
        </item>
    </argument>
</fieldset>
```
* Класс [`Magento\Ui\Component\Form\Fieldset`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Form/Fieldset.php)
* JS модуль [`fieldset.js`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/view/base/web/js/form/components/fieldset.js)

Пример конфигурации для клиента:
```xml
<fieldset name="customer">
        <settings>
            <label translate="true">Account Information</label>
        </settings>
        <field name="entity_id" formElement="input">
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="source" xsi:type="string">customer</item>
                </item>
            </argument>
            <settings>
                <dataType>text</dataType>
                <visible>false</visible>
            </settings>
        </field>
    </fieldset>
```

Внутри fieldset можно создавать группы для группировки нескольких полей:
```xml
<container name="container_group" component="Magento_Ui/js/form/components/group" sortOrder="20">
    <argument name="data" xsi:type="array">
        <item name="type" xsi:type="string">group</item>
        <item name="config" xsi:type="array">
            <item name="dataScope" xsi:type="boolean">false</item>
            <item name="validateWholeGroup" xsi:type="boolean">true</item>
        </item>
    </argument>
    <field name="group_id" formElement="select">
        <argument name="data" xsi:type="array">
            <item name="config" xsi:type="array">
                <item name="fieldGroup" xsi:type="string">group_id</item>
                <item name="source" xsi:type="string">customer</item>
            </item>
        </argument>
        <settings>
            <required>true</required>
            <dataType>number</dataType>
        </settings>
    </field>
    <field name="disable_auto_group_change" formElement="checkbox" class="Magento\Customer\Ui\Component\Form\Field\DisableAutoGroupChange">
        <argument name="data" xsi:type="array">
            <item name="config" xsi:type="array">
                <item name="fieldGroup" xsi:type="string">group_id</item>
                <item name="source" xsi:type="string">customer</item>
                <item name="default" xsi:type="number">0</item>
            </item>
        </argument>
        <settings>
            <dataType>boolean</dataType>
        </settings>
        <formElements>
            <checkbox>
                <settings>
                    <description translate="true">Disable Automatic Group Change Based on VAT ID</description>
                    <valueMap>
                        <map name="false" xsi:type="string">0</map>
                        <map name="true" xsi:type="string">1</map>
                    </valueMap>
                    <prefer>checkbox</prefer>
                </settings>
            </checkbox>
        </formElements>
    </field>
</container>
```

`container` в `definition.xml`:
```xml
<container class="Magento\Ui\Component\Container" component="uiComponent">
    <argument name="data" xsi:type="array">
        <item name="template" xsi:type="string">templates/container/default</item>
    </argument>
</container>
```
* Класс [`Magento\Ui\Component\Container`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Container.php)

### Элементы форм

Все доступные элементы форм можно посмотреть [здесь](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/view/base/ui_component/etc/definition.xml#L112)

Доступные типы данных:

* text
* file
* number
* date
* price
* image
* email
* boolean

Все они наследуются от [`Magento\Ui\Component\Form\Element\DataType\AbstractDataType`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Form/Element/DataType/AbstractDataType.php)

`input` в `definition.xml`:
```xml
<input class="Magento\Ui\Component\Form\Element\Input" component="Magento_Ui/js/form/element/abstract" template="ui/form/field"/>
```
* Класс [`abstract.js`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/Component/Form/Element/Input.php)
* JS модуль [`fieldset.js`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/view/base/web/js/form/element/abstract.js)
* Темплейт [field.html](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Ui/view/base/web/templates/form/field.html)

Пример конфигурации поля input:
```xml
<field name="prefix" formElement="input">
    <argument name="data" xsi:type="array">
        <item name="config" xsi:type="array">
            <item name="source" xsi:type="string">customer</item>
        </item>
    </argument>
    <settings>
        <dataType>text</dataType>
        <visible>true</visible>
    </settings>
</field>
```
