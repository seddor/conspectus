# Determine the layout initialization process

>Determine how layout is compiled. How would you debug your `layout.xml` files and verify that the right layout instructions are used?

>Determine how HTML output is rendered. How does Magento flush output, and what mechanisms exist to access and customize output?

>Determine module layout XML schema. How do you add new elements to the pages introduced by a given module?

>Demonstrate the ability to use layout fallback for customizations and debugging. How do you identify which exact `layout.xml` file is processed in a given scope? How does Magento treat layout XML files with the same names in
different modules?

>Identify the differences between admin and frontend scopes. What differences exist for layout initialization for the admin scope?

[Layout overview](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/layout-overview.html)

## Компиляция

* [`Magento\Framework\View\Layout::build()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout.php#L256)
* [`Magento\Framework\View\Layout\Builder::build()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout/Builder.php#L59)
* [`loadLayoutUpdates()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout/Builder.php#L75)
  * генерирует событие `layout_load_before`
* [`generateLayoutXml()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout/Builder.php#L98)
* [`generateLayoutBlocks()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout/Builder.php#L116)
  * генерирует событие `layout_generate_blocks_before`
  * генерирует событие  `layout_generate_blocks_after`

## Вывод 

Происходит в методе [`Magento\Framework\View\Layout::getOutput()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout.php#L958)

## Процесинг

M2 процесит файлы лаяутов в таком порядке:

1. Загружаются `base` файлы модулей
2. Загружаются `area-специфичные` файлы модулей
3. Собираются все файлы лаяутов из модулей, в порядке в котром модули указаны в `app/etc/module.php`
4. Определяется [иерархия тем](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/themes/theme-inherit.html) 
5. Файлы преопределяют друг-друга в соответстивии с иерархией
6. Все файлы мерджатся

## Extending

[Extend a layout](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/layout-extend.html)

Для расширения/измения лаяута родительской темы можно создать extending файл.

распологается он в:

* для page configuration и generic layout:
```
<theme_dir>
 |__/<Namespace>_<Module>
   |__/layout
     |--<layout1>.xml
     |--<layout2>.xml
```
* для page layout:
```
<theme_dir>
 |__/<Namespace>_<Module>
   |__/page_layout
     |--<layout1>.xml
     |--<layout2>.xml
```

в конечном итоге файлы будут смерджены так:
```xml
<layouts xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <handle id="checkout_cart_index" label="Shopping Cart" type="page" parent="default">
        <!-- Layout instructions from checkout_cart_index.xml -->
    </handle>
    <handle id="checkout_onepage_index" label="One Page Checkout" type="page" parent="default">
        <!-- Layout instructions from checkout_onepage_index.xml -->
    </handle>
    <!-- ... -->
</layouts>
```

## Overriding

[Override a layout](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/layout-override.html)

Если лаяут необходимо переопределить полность исользуется оверрайдинг.
При его использовании вместо файла из родительской темы берётся перезаписывающий файл.

Переопределения нужно когда нужно:
* Удаление аргуметов методов
* Изменение/удаление подключенных хендлов через инструкции `<update handle="" />`
* удаление всех элементов лаяута путём определения пустого лаяута.

Распложение:

* Для модуля:
```
<theme_dir>
  |__/<Namespace_Module>
    |__/layout
      |__/override
         |__/base
           |--<layout1>.xml
           |--<layout2>.xml
```
* Для темы:
```
<theme_dir>
  |__/<Namespace_Module>
    |__/layout
      |__/override
         |__/theme
            |__/<Parent_Vendor>
               |__/<parent_theme>
                  |--<layout1>.xml
                  |--<layout2>.xml
```

Использование переопределения может вызвать последствие:
* при изменении имени или алиаса блока ссылки на него перестанут работать
* при обновлении переопределённого лаяута не применяются новые измения

## Кастомизация в админке

Неоторые сущности могут иметь индивидуальный лаяут, другую тему и тип страницы.

Сущности:
* Продукт
* Категория
* CMS-страница

Допустимые инструкции:
* referenceContainer
* container
* update
* move
* head
* body

## Разница лаяутов на фронте и в админке

Лаяуты в админке отключаются:

* в админке автоматически инциализируется блок с сообщениями, в том числе для кастомных блоков
* кастомный абстрактный блок [`Magento\Backend\Block\AbstractBlock`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/Block/AbstractBlock.php)
  * включает `AuthorizationInterface`
* кастомный блок с темплейтом [`Magento\Backend\Block\Template`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/Block/Template.php)
  * генерирует ивенты `adminhtml_block_html_before` для некешированных блоков и `view_block_abstract_to_html_after` для кешированных.
* кастомный объект резльтата выполнения контроллера: [`Magento\Backend\Model\View\Result\Page`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/Model/View/Result/Page.php)

## Дополнительно

[Common layout customization tasks](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/xml-manage.html)
[Customizing layout illustration](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/layout-practice.html)
