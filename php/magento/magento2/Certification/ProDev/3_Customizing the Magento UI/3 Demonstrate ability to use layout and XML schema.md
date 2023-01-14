# Demonstrate ability to use layout and XML schema

>Describe the elements of the Magento layout XML schema, including the major XML directives. How do you use layout XML directives in your customizations?

>Describe how to create a new layout XML file.

>Describe how to pass variables from layout to block.

[Using Layout and XML Schema in Magento 2](https://belvg.com/blog/using-layout-and-xml-schema-in-magento-2.html)
[Layout overview](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/layout-overview.html)

В M2 основными компонентами дизайна страницы являются лаяуты, контейнеры и блоки. Лаяут описывает структуру страницы, контейнер предоставляет плейсхолдеры в структуре этой страницы, которые заполняются блоками.

Как и в M1 лаяуты описываются в xml файлах, однако, есть некоторые измения: добавлены разные типы лаяутов:

* Page layout — используется для создания и конфигурации типа страницы (одноколоночная, двух, трёх и т.п.)
* Page configuration — используется для создания и конфигурации целевой страницы и её частей
* Generic layout — как и Page configuration, в основном используется для ajax-запросов.

В M2 файлы лаяутов именуются:
* Page layout — по названию типа страницы, например `1column.xml`
* Page configuration и Generic layout — по имени хендла, `default` — применяется для всех страниц.

## Расположение

Лаяуты могут находиться в этих директориях (по приоритету):

* _`<theme_dir>`/`<Namespace_Module>`/layout/override/base/theme/`<Namespace>`/`<theme>`_
* _`<theme_dir>`/`<Namespace_Module>`/layout/override/base`_
* _`<theme_dir>`/view/frontend/layout_
* _`<module_dir>`/view/frontend/layout_
* _`<module_dir>`/view/base/layout_

Для Page layout список аналогичный, только вместо `layout` будет `page_layout`.

## Типы лаяутов

[Layout file types](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/layout-types.html)

### Page layout

Корневая нода `layout`, файл использует схему `urn:magento:framework:View/Layout/etc/page_layout.xsd`

Доступные инструкции:

* [`<container>`](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/xml-instructions.html#fedg_layout_xml-instruc_ex_cont) — конструкция, создающая контейнер в структуре дерева страницы. Контейнер предназначен для хранения блоков или других контейнеров и их рендеринга без какой-либо функциональности со своей стороны. В единственном обязательном аргументе name указывается имя элемента (уникальное на всю страницу), в `as` — алиас (уникальный среди соседних элементов внутри общего родителя). Атрибуты `before` и `after` предназначены для сортировки блока среди соседних блоков внутри общего родителя. Возможно использование в качестве тега-обертки, через аргументы `htmlTag`, `htmlClass` и `htmlId`. Аналог структурных типов блоков в M1.
* [`<referenceContainer>`](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/xml-instructions.html#fedg_layout_xml-instruc_ex_ref) — конструкция с обязательным атрибутом name, указывающим на какой-либо контейнер и позволяющая добавлять к нему потомков. Атрибут `remove` позволяет полностью удалить контейнер, а `display` - спрятать (т.е. не рендерить в html-код страницы, но сохраняя всю функциональность).
* [`<move>`](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/xml-instructions.html#fedg_layout_xml-instruc_ex_mv) — позволяет перенести элемент указанный в аргументе `element` внутрь другого элемента, указанного в аргументе `destination`. Также, доступны атрибуты `as` - для смены алиаса, и `before/after` для сортировки.
* [`<update>`](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/xml-instructions.html#fedg_layout_xml-instruc_ex_upd) — элемент верхнего уровня, позволяет использовать другой хэндл, указанный в аргументе `handle` подлючая все его конструкции.

Пример(2columns-left.xml):
```xml
<layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_layout.xsd">
    <update handle="1column"/>
    <referenceContainer name="columns">
        <container name="div.sidebar.main" htmlTag="div" htmlClass="sidebar sidebar-main" after="main">
            <container name="sidebar.main" as="sidebar_main" label="Sidebar Main"/>
        </container>
        <container name="div.sidebar.additional" htmlTag="div" htmlClass="sidebar sidebar-additional" after="div.sidebar.main">
            <container name="sidebar.additional" as="sidebar_additional" label="Sidebar Additional"/>
        </container>
    </referenceContainer>
</layout>
```

Для использования кастомного Page layout его нужно определить в `layouts.xml`, этот файл может находиться в:

* _`<module_dir>`/view/frontend/layouts.xml_
* _`<module_dir>`/view/base/layouts.xml_
* _`<theme_dir>`/`<Namespace>_<Module>`/layouts.xml_

Пример:
```xml
<page_layouts xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/PageLayout/etc/layouts.xsd">
    <layout id="3columns">
        <label translate="true">3 columns</label>
    </layout>
</page_layouts>
```

для использования Page layout в Page configuration нужно указать его в атрибуте `layout` ноды `page`:
```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" layout="3columns" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <update handle="customer_account"/>
    <body>
        <referenceContainer name="content">
            <block class="Magento\Wishlist\Block\Customer\Sharing" name="wishlist.sharing" template="Magento_Wishlist::sharing.phtml" cacheable="false"/>
        </referenceContainer>
    </body>
</page>
```

### Page configuration

Корневая нода — `<page>`

так-же может содержать:

* `html` — содержит атрибты html страницы
* `head` — соответствует `head` html страницы, содержит:
  * `<title>` — заголовок страницы
  * `<meta>` — мета заголовок
  * `<link>` — инструкция для подключения статики, имеет атрибуты:
    * defer
    * ie_condition
    * charset
    * hreflang
    * media
    * rel
    * rev
    * sizes
    * src
    * src_type
    * target
    * type
  * `<css>` — для подключеия css, атрибуты такие-же как у `link`
  * `<script>` — для подключения js, атрибуты:
    * defer
    * ie_condition
    * async
    * charset
    * src
    * src_type
    * type
* `body` — содержит описание блочной структуры, может содержать такие инструкции:
  * block
  * container
  * move
  * attribute
  * referenceBlock
  * referenceContainer
  * action

### Generic Layout

Описывает только body страницы

Корневая нода — layout

* update — подключает какой-либо хендл
* `<container name=”root”>` — обязательный элемент, корневой контейнер, содержит:
  * block
  * container
  * referenceBlock
  * referenceContainer

Пример:
```xml
<layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/layout_generic.xsd">
    <update handle="formkey"/>
    <update handle="adminhtml_googleshopping_types_block"/>
    <container name="root">
        <block class="Magento\Backend\Block\Widget\Grid\Container" name="googleshopping.types.container" template="Magento_Backend::widget/grid/container/empty.phtml"/>
    </container>
</layout>
```

## Инструкции лаяутов

[Layout instructions](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/xml-instructions.html)

## Передача данных в блоки

В блоках все параметры переаднные ему в `<arguments>` в лаяуте попадют в итоге в `$_data` поле блока, соответственно для них в темлейте и блоке можно использовать гетеры и сетеры.

## Контейнеры обрачивающие элменты в теги

```xml
<container name="my.container" htmlTag="div">
    <!-- ... -->
</container>
```
Так-же есть атрибуты:
* `htmlId`
* `htmlClass`
