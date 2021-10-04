# Define form and grid widgets

>Define form structure, form templates, grids, grid containers, and elements. What steps are needed to display a grid or form?
>Describe the grid and form workflow. How is data provided to the grid or form? How can this be process be customized or extended?
>Describe how to create a simple form and grid for a custom entity. Given a specific entity with different types of fields (text, dropdown, image, file, date, and so on) how would you create a grid and a form

Для форм и гридов в M2 используется UI компоненты (UI components).

## Define form structure, form templates, grids, grid containers, and elements. What steps are needed to display a grid or form?

UI компоненты имеют свои XML-файлы настроек, они находятся в _`<module-dir>`/view/adminhtml/ui\_component_, для валидации используется схема: [`urn:magento:module:Magento_Ui:etc/ui_configuration.xsd`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Ui/etc/ui_configuration.xsd).

### Grid

[Admin Grids](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/admin-grid.html)

1. Нужно добавить UI компонент в хендл страницы. Хендл находится в файле лаяута страницы в _`<module-dir>`/view/adminhtml/layout/`<handle_name>`.xml_, содержимое:
```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceContainer name="content">
            <uiComponent name="<grid_component_name>"/>
        </referenceContainer>
    </body>
</page>
```
2. Нужно настроить грид, настройки будут находиться в _`<module-dir>`/view/adminhtml/ui\_component/`<grid_component_name>`.xml_, содержимое:
```xml
<listing xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Ui:etc/ui_configuration.xsd">
    <argument name="data" xsi:type="array">
        <item name="js_config" xsi:type="array">
            <item name="provider" xsi:type="string">oggetto_skin_care_mgm_link_listing.oggetto_skin_care_mgm_link_listing_data_source</item>
        </item>
    </argument>
    <settings>
        <spinner>oggetto_skin_care_mgm_link_listing_columns</spinner>
        <deps>
            <dep>oggetto_skin_care_mgm_link_listing.oggetto_skin_care_mgm_link_listing_data_source</dep>
        </deps>
    </settings>
    <dataSource name="oggetto_skin_care_mgm_link_listing_data_source" component="Magento_Ui/js/grid/provider">
        <settings>
            <storageConfig>
                <param name="indexField" xsi:type="string">entity_id</param>
            </storageConfig>
            <updateUrl path="mui/index/render"/>
        </settings>
        <aclResource>Oggetto_SkinCareMgmLink::skin_care_mgm_link</aclResource>
        <dataProvider class="Magento\Framework\View\Element\UiComponent\DataProvider\DataProvider" name="oggetto_skin_care_mgm_link_listing_data_source">
            <settings>
                <requestFieldName>id</requestFieldName>
                <primaryFieldName>entity_id</primaryFieldName>
            </settings>
        </dataProvider>
    </dataSource>
    <listingToolbar name="listing_top">
        <bookmark name="bookmarks"/>
        <columnsControls name="columns_controls"/>
        <exportButton name="export_button"/>
        <filters name="listing_filters">
            <settings>
                <templates>
                    <filters>
                        <select>
                            <param name="template" xsi:type="string">ui/grid/filters/elements/ui-select</param>
                            <param name="component" xsi:type="string">Magento_Ui/js/form/element/ui-select</param>
                        </select>
                    </filters>
                </templates>
            </settings>
        </filters>
        <paging name="listing_paging"/>
        <massaction name="listing_massaction" component="Magento_Ui/js/grid/tree-massactions">
            <action name="delete">
                <settings>
                    <confirm>
                        <message translate="true">Delete selected items?</message>
                        <title translate="true">Delete items</title>
                    </confirm>
                    <url path="inventory/stock/massDelete"/>
                    <type>delete</type>
                    <label translate="true">Delete</label>
                </settings>
            </action>
        </massaction>
    </listingToolbar>
    <columns name="oggetto_skin_care_mgm_link_listing_columns">
        <selectionsColumn name="ids">
            <settings>
                <indexField>entity_id</indexField>
            </settings>
        </selectionsColumn>
        <column name="id">
            <settings>
                <filter>textRange</filter>
                <label translate="true">ID</label>
                <sorting>asc</sorting>
            </settings>
        </column>
        <column name="email">
            <settings>
                <filter>text</filter>
                <label translate="true">Email</label>
            </settings>
        </column>
        <column name="is_fired" component="Magento_Ui/js/grid/columns/select">
            <settings>
                <options class="Magento\Config\Model\Config\Source\Yesno"/>
                <filter>select</filter>
                <dataType>select</dataType>
                <label translate="true">Is Fired</label>
            </settings>
        </column>
        <column name="created_at" class="Magento\Ui\Component\Listing\Columns\Date" component="Magento_Ui/js/grid/columns/date">
            <settings>
                <filter>dateRange</filter>
                <dataType>date</dataType>
                <label translate="true">Created At</label>
            </settings>
        </column>
    </columns>
</listing>
```

Основное:

* argument[data]/item[js_config]/item[provider] — указывает источник данных для грида, здесь это oggetto_skin_care_mgm_link_listing.oggetto_skin_care_mgm_link_listing_data_source:
  * oggetto_skin_care_mgm_link_listing — имя грида, соответствует имени файла конфига и указывается в хендле лаяута.
  * oggetto_skin_care_mgm_link_listing_data_source имя dataProvider класса предоставляющего данные, это имя задаётся в секции dataSource/dataProvider[name]
* settings/spinner — указывается имя копонента-колонок, после загрузки которого спиенер скроется
* dataSource — настройка источника данных
  * aclResource — ACL-ресурс используемый для грида
  * dataProvider — PHP-класс отвечающий за обработку и получение данных. Data provider класс должен реализовывать интерфейс [`Magento\Framework\View\Element\UiComponent\DataProvider\DataProviderInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/UiComponent/DataProvider/DataProviderInterface.php) или наследовать [`Magento\Ui\DataProvider\AbstractDataProvider`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Ui/DataProvider/AbstractDataProvider.php), сам класс занимается получением данных, добавлением фильтров, сортировок, работой с пагинацией и т.д.. Для стандартного грида можно использовать класс [`Magento\Framework\View\Element\UiComponent\DataProvider\DataProvider`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/UiComponent/DataProvider/DataProvider.php)
  * updateUrl — урл для обновления грида, обычно всегда используется стандартный: mui/index/render
* listingToolbar — содержт различные элементы грида, например:
  * bookmark — позволяет сохранять различные настройки отображения грида
  * columnsControls — позволяет управлять отображением колонок на гриде
  * filters — включает отображение фильтров, здесь настраиваются параметры рендеринга фильтров, сами фильтры настраиваются в колонках
  * exportButton — кнопки для экспорта данных грида
  * paging — пагинация
  * massaction — массовые действия над колонками, например удаление нескольких колонок
* columns — содержит настройку колонок грида
  * column[name] — имя соответствует полю в таблице
  * column[class] — для колонки можно указать кастомный класс. Стандартный класс [`Magento\Ui\Component\Listing\Columns\Column`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Ui/Component/Listing/Columns/Column.php) он реализует [`Magento\Ui\Component\Listing\Columns\ColumnInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Ui/Component/Listing/Columns/ColumnInterface.php), в классе можно изменить выводимые данные и применеть сортировки для колонки.
  * column[component] — для колонки можно указать кастомный js-компонент
  * column/settings/filter — тип фильтра, всего, по-умолчанию, доступно [5 типа фильтров](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Ui/Component/Filters.php#L37-L41):
    * text
    * textRange
    * select
    * dateRange
    * datetimeRange
  * column/settings/dataType — тип данных

3. Настроить (и создать) классы для грида:
  * `di.xml`:
```xml
    <type name="Magento\Framework\View\Element\UiComponent\DataProvider\CollectionFactory">
        <arguments>
            <argument name="collections" xsi:type="array">
                <item name="oggetto_skin_care_mgm_link_listing_data_source" xsi:type="string">Oggetto\SkinCareMgmLink\Model\ResourceModel\Link\Grid\Collection</item>
            </argument>
        </arguments>
    </type>
    <virtualType name="Oggetto\SkinCareMgmLink\Model\ResourceModel\Link\Grid\Collection" type="Magento\Framework\View\Element\UiComponent\DataProvider\SearchResult">
        <arguments>
            <argument name="mainTable" xsi:type="string">skin_care_mgm_link</argument>
            <argument name="resourceModel" xsi:type="string">Oggetto\SkinCareMgmLink\Model\ResourceModel\Link</argument>
        </arguments>
    </virtualType>
```
  * в конфиге UI компонента:
```xml
<dataProvider class="Magento\Framework\View\Element\UiComponent\DataProvider\DataProvider" name="oggetto_skin_care_mgm_link_listing_data_source">
```
В зависимости от реализации может понадобится указать только dataProvider.

### Form

[https://www.mageplaza.com/devdocs/creat-a-ui-form-in-magento-2.html](https://www.mageplaza.com/devdocs/creat-a-ui-form-in-magento-2.html)

1. Нужно добавить UI компонент в хендл страницы. Хендл находится в файле лаяута страницы в _`<module-dir>`/view/adminhtml/layout/`<handle_name>`.xml_, содержимое:
```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <update handle="styles"/>
    <body>
        <referenceContainer name="content">
            <uiComponent name="<from_component_name>"/>
        </referenceContainer>
    </body>
</page>
```
2. Нужно настроить форму, настройки будут находиться в _`<module-dir>`/view/adminhtml/ui\_component/`<from_component_name>`.xml_, содержимое:
```xml
<form xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Ui:etc/ui_configuration.xsd">
    <argument name="data" xsi:type="array">
        <item name="js_config" xsi:type="array">
            <item name="provider" xsi:type="string">cms_block_form.block_form_data_source</item>
        </item>
        <item name="label" xsi:type="string" translate="true">General Information</item>
        <item name="template" xsi:type="string">templates/form/collapsible</item>
    </argument>
    <settings>
        <buttons>
            <button name="save" class="Magento\Cms\Block\Adminhtml\Block\Edit\SaveButton"/>
            <button name="delete" class="Magento\Cms\Block\Adminhtml\Block\Edit\DeleteButton"/>
            <button name="back" class="Magento\Cms\Block\Adminhtml\Block\Edit\BackButton"/>
        </buttons>
        <namespace>cms_block_form</namespace>
        <dataScope>data</dataScope>
        <deps>
            <dep>cms_block_form.block_form_data_source</dep>
        </deps>
    </settings>
    <dataSource name="block_form_data_source">
        <argument name="data" xsi:type="array">
            <item name="js_config" xsi:type="array">
                <item name="component" xsi:type="string">Magento_Ui/js/form/provider</item>
            </item>
        </argument>
        <settings>
            <submitUrl path="cms/block/save"/>
        </settings>
        <dataProvider class="Magento\Cms\Model\Block\DataProvider" name="block_form_data_source">
            <settings>
                <requestFieldName>block_id</requestFieldName>
                <primaryFieldName>block_id</primaryFieldName>
            </settings>
        </dataProvider>
    </dataSource>
    <fieldset name="general">
        <settings>
            <label/>
        </settings>
        <field name="block_id" formElement="input">
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="source" xsi:type="string">block</item>
                </item>
            </argument>
            <settings>
                <dataType>text</dataType>
                <visible>false</visible>
                <dataScope>block_id</dataScope>
            </settings>
        </field>
        <field name="is_active" sortOrder="10" formElement="checkbox">
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="source" xsi:type="string">block</item>
                    <item name="default" xsi:type="number">1</item>
                </item>
            </argument>
            <settings>
                <dataType>boolean</dataType>
                <label translate="true">Enable Block</label>
                <dataScope>is_active</dataScope>
            </settings>
            <formElements>
                <checkbox>
                    <settings>
                        <valueMap>
                            <map name="false" xsi:type="number">0</map>
                            <map name="true" xsi:type="number">1</map>
                        </valueMap>
                        <prefer>toggle</prefer>
                    </settings>
                </checkbox>
            </formElements>
        </field>
        <field name="title" sortOrder="20" formElement="input">
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="source" xsi:type="string">block</item>
                </item>
            </argument>
            <settings>
                <validation>
                    <rule name="required-entry" xsi:type="boolean">true</rule>
                </validation>
                <dataType>text</dataType>
                <label translate="true">Block Title</label>
                <dataScope>title</dataScope>
            </settings>
        </field>
    </fieldset>
</form>
```
Основное:
  * settings/buttons — задаёт кнопки на форме, каждая кнопка это отдельный класс реализующий [`Magento\Framework\View\Element\UiComponent\Control\ButtonProviderInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/UiComponent/Control/ButtonProviderInterface.php)
  * dataSource
    * settings/submitUrl — урл сабмита формы
    * dataProvider — класс предоставляющий данные, здесь всё аналогично гридам
  * fieldset — компонет набора полей, форма может содержать несколько таких компонентов с разным name
    * field — компонент поля формы

## Describe the grid and form workflow. How is data provided to the grid or form? How can this be process be customized or extended?

### Флоу генерации

1. [`Magento\Framework\View\Layout\Generator\UiComponent::generateComponent()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout/Generator/UiComponent.php#L120) — создаёт и готовит копоненты
2. [`Magento\Framework\View\Element\UiComponentFactory::create()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/UiComponentFactory.php#L209)  — мерджит метаданные, создаёт дочерние копоненты
3. [`Magento\Ui\Component\Wrapper\UiComponent`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Ui/Component/Wrapper/UiComponent.php) — выводит html копонентов, затем они рендарятся уже через JS.

### Получение данных

Данные для копнонентов получаеются из dataProvider — PHP-класс отвечающий за обрботку и получение данных. Data provider класс должен реализовывать интерфейс [`Magento\Framework\View\Element\UiComponent\DataProvider\DataProviderInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/UiComponent/DataProvider/DataProviderInterface.php) или наследовать [`Magento\Ui\DataProvider\AbstractDataProvider`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Ui/DataProvider/AbstractDataProvider.php). Data Provider настраивается в разделе `dataSource` компонента.

### Кастомизации

* Колонки грида — поддерживают атрибут class, в котором можно указать кастомный класс для колонки, он должен реализовывать [`Magento\Ui\Component\Listing\Columns\ColumnInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Ui/Component/Listing/Columns/ColumnInterface.php)
* поля формы — поддерживает атрибут class, в котором можно указать кастомный класс для поля, он должен наследовать [`Magento\Ui\Component\Form\Field`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Ui/Component/Form/Field.php)
* фильтры в гриде — допустимость кастомизации фильтров в гриде зависит от dataProvider, он должен поддерживать модификацию фильтров для кастомизации. Примеры:
  * [`Magento\Catalog\Ui\DataProvider\Product\ProductDataProvider`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Ui/DataProvider/Product/ProductDataProvider.php) — продуковый грид, поддерживает кастомизацию фильтров для полей, кастомизирующий класс должен реализовывать [`Magento\Ui\DataProvider\AddFieldToCollectionInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Ui/DataProvider/AddFilterToCollectionInterface.php)
  * [`Magento\Cms\Ui\Component\DataProvider`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Cms/Ui/Component/DataProvider.php) — грид CMS сущностей, содержит свою реализацию повзволяющую добавлять кастомные фильтры, фильтр должен реализовывать [`Magento\Cms\Ui\Component\AddFilterInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Cms/Ui/Component/AddFilterInterface.php)
