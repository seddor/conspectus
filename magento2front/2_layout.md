# Layout

Лаяуты в M2 претерпели некоторые измения по сравнению с M1.

Типы лаяутов:
* page layout — используется для создания и конфигурации типа страницы (одноколоночная, двух, трёх и т.п)
* Page configuration — используется для создания и конфигурации целой страницы или её части
* Generic Layout — аналогичен page configuration, но используется обычно для ajax-запросов.

page layout называются по типу страницы, например `1column.xml`, остальные типы по имени хендала и содержат только инструкции для этого хендла, например `catalog_product_view.xml`.

Часто используемые хендлы:
* default - хэндл по умолчанию, применяется для всех страниц;
* default_head_blocks - подключается на всех страницах, используется для подключения ресурсов в <head>;
* cms_index_index - хэндл главной страницы;
* сatalog_product_view - хэндл страницы продукта;
* catalog_product_view_type_simple (либо configurable, downloadable, virtual, bundle или grouped) - хэндл страницы продукта определенного типа;
* catalog_category_view - хэндл страница категории;
catalog_category_view_type_layered - хэндл страница категории с фильтром;
* catalog_category_view_type_default - хэндл страницы категории без фильтра;
* checkout_cart_index - хэндл страницы корзины;
* checkout_index_index - хэндл страницы чекаута;
* checkout_onepage_success - хэндл страницы “спасибо за покупку”.

Каталоги лаяутов(по приоритету):
1. `<theme_dir>/<Namespace_Module>/layout/override/theme/<Namespace>/<Theme>`
2. `<theme_dir>/<Namespace_Module>/layout/override/base`
3. `<theme_dir>/<Namespace_Module>/layout`
4. `<module_dir>/view/frontend/layout`
5. `<module_dir>/view/base/layout`

Для page layout всё аналогично, только layout заменяется на page_layout.

## Page layout

Предназначены для созадния конфигурации типа страниц. 
Корневой элемент XML-файла - `<layout>`, файл должен следовать схеме urn:magento:framework:View/Layout/etc/page_layout.xsd
Внутри разрешабтся такие ноды:
* `<update>` — элемент верхнего уровня, позволяет использовать другие хендл:
```xml
<update handle="empty"/>
```
* `<container>`­ — создаёт контейнер в структуре дерева страницы. Контейнер предназначен для хранения блоков или дркгих контейнеров и их редеринга. 
Аргументы:
    * name — имя, уникально на всю страницу
    * as — алиас, уникальный среди соседних элементов внутри общего родителя
    * before/after — для сортировки среди соседних блоков, внутри общего родителя. 
* `<referenceContainer>` ­— референс к какому-либо контейнеру, позволяет добавлять потомков. 
    * remove — удалит контейнер
    * display — скрывает контейнер(не рендерится, но функциональность сохраняется)
* `<move>` — позволяет переносить элемент внутрь друго.

После написания лаяута для нового типа страницы его необходимо зарегистрировать в `layouts.xml`, он может находиться в:
1. `<module_dir>/view/frontend/layouts.xml`
2. `<module_dir>/view/base/layouts.xml`
3. `<theme_dir>/<Namespace>_<Module>/layouts.xml`

```xml
<page_layouts xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/PageLayout/etc/layouts.xsd">
    <layout id="1column">
        <label translate="true">1 column</label>
    </layout>
</page_layouts>
```

## Page configuration

Используется для создания и конфигурации страницы или её части. 

Лаяуты располагаются в:
* `<module_dir>/view/frontend/layout`
* `<theme_dir>/<Namespace>_<Module>/layout`

Особые теги:
* `<page></page>` – корневой элемент page configuration layout, имеет атрибут layout, в котором задается page layout для страницы.
* `<html>` – может содержать внутри себя инструкции  `<attribute name=”name” value=”value”>`, что превращается в `<html name=”value” …>` html-страницы.
* `<head>` – соответствует одноименному HMTL-тегу, содержит `<title>`, `<meta>`, `<link>`, `<css>`, `<script>`.
* `<title>` – meta-title страницы
* `<meta>` – HTML–тег meta, имеет атрибуты content, charset, http-equiv, name, scheme
* `<link>` – инструкция для подключения файла статики: css, js. Имеет следующие атрибуты:
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
* `<css>` – инструкция для подключения css-файлов, атрибуты аналогичны предыдущей инструкции
* `<script>` – инструкция для подключения js-файлов, имеет только следующие атрибуты:
    * defer
    * ie_condition
    * async
    * charset
    * src
    * src_type
    * type
* `<body>` – внутри данной инструкции содержится описание блочной структуры страницы, разрешенные инструкции внутри:
    * `<block>`
    * `<container>`
    * `<move>`
    * `<attribute>`
    * `<referenceBlock>`
    * `<referenceContainer>`
    * `<action>`

## Generic layout

Содержит только описания содержимого `<body>`

Находится в:
1. `<module_dir>/view/frontend/layout`
2. `<theme_dir>/<Namespace>_<Module>/layout`

Содержит:
* `<layout>` – корневой элемент
* `<update>` – подключает какой-либо хэндл лэйаута
* `<container name=”root”>` – обязательный элемент, корневой контейнер, содержит:
    * `<block>`
    * `<container>`
    * `<referenceBlock>`
    * `<referenceContainer>`

## Fallback, rewriting & extending

## extending

Лаяуты мерджатся по имени, т.е. по хендлу

## Overriding

Можно переопределять лаяуты, тогда вместо лаяуту из родительской темы, в качестве основного будет использоваться файл из текущей темы.

Когда нужно переопределение:
* Удаление аргументов методов
* Изменение или удаление подключения хендлов через инструкцию `<update>`
* Удаление всех элементов лаяута, при переодпределнии лаяута файлом с пустым содержимым.

Переопределения для модулей:
`<theme_dir>/<Namespace_Module>/layout/override/base/<layout1>.xml`
Для темы:
`<theme_dir>/<Namespace_Module>/layout/override/theme/<Parent_Vendor>/<parent_theme>/<layout1>.xml`

Последставия:
* При измении имени или алиаса блока ссылки на него становятся недоступными
* При обновлении переопределённого лайяута, не применяются измения.
