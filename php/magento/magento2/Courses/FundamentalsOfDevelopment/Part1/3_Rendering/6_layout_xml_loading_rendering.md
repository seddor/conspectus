# Лаяут: загрузка и рендеринг

## area 

лаяуты подгружаются для таких area:

* base — лаяуты из base примянятся сразу на adminhtml и frontend. Эта area доступна только в директории модулей, в теме её использовать нельзя.
* adminhtml
* frontend

## Page layout

Базовый лаяут страницы указывается в `layout` атрибуту рутовой ноды `<page>`. Значения берутся из подиректори page_layout директорий _view/`<area>`_.

В page layout определяется базовая структура страницы.

## Наследование в темах

При загрузке XML файлов, M2 иерархично применяет темы, до тех пор пока не дойдёт до темы без родителя.

Наслеодвание настраивается в `theme.xml` или в `composer.json` файлах темы. Как правило базовой темой в M2 является `Magento/blank`.

Метод в котором происохдит поиск родительских темы: [`Magento\Theme\Model\Theme::getInheritedThemes()`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Theme/Model/Theme.php#L373)

## Overriding лаяутов

Тема может польность заменить лаяут из родительской темы, путем размещения файла-замены в дректоии _layout/override/`<area>/*.xml`_ тоже самое можно сделать для page layout. Например: _app/design/frontend/Training/custom/Magento\_Catalog/layout/override/theme/Magento/blank/catalog\_product\_view.xml_ заменяет `catalog_product_view.xml` из темы Magento_Blank.

Overriding файлов темы рекомендуется избрегать, т.к. он может повлечь за собой проблемы пряпятсвующие последующим обновленим лаяута.

# Хендлы

## default

В большинство страниц в мадженто используется хендл `default`. Он используется для настройки основных контейнеров и элементов, распологающихся на каждой странице. `default` всегда обрабатывается первым.

## Странице-специфичные экшен хендлы

Странице-специфичные экшен хендлы ассоциируются с конкретной страницей, например `catalog_product_view.xml` для _catalog/product/view_.

## update

Так же существует хендлы для различных специфчных случаев, например:

* `cms_index_index_id_home`
* `catalog_product_prices`
* `catalog_product_view_id_920`

## custom

Есть несколько способов добавить хендлы:

* методом [`Magento\Framework\View\Result\Layout::addHandle()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/View/Result/Layout.php#L138) для объекта результата.
* через лаяут XML — `<update handle="custom_handle"/>`
