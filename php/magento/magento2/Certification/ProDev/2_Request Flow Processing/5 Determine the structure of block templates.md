# Determine the structure of block templates

>Identify and understand root templates, `empty.xml`, and `page_layout`. How are page structures defined, including number of columns, which basic containers are present, etc.?

>Describe the role of blocks and templates in the request flow. In which situations would you create a new block or a new template?

[How to Work with the Structure of Block Templates, Root Templates, empty.xml, and page_layout in Magento 2](https://belvg.com/blog/working-with-block-templates-root-templates-empty-xml-and-page_layout-in-m2.html)

## Базовые Page layouts

* base
  * [`empty.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Theme/view/base/page_layout/empty.xml)
* frontend
  * [`1column.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Theme/view/frontend/page_layout/1column.xml)
  * [`2columns-left.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Theme/view/frontend/page_layout/2columns-left.xml)
  * [`2columns-left.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Theme/view/frontend/page_layout/2columns-right.xml)
  * [`3columns.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Theme/view/frontend/page_layout/3columns.xml)
* adminhtml
  * [`admin-1column.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Theme/view/adminhtml/page_layout/admin-1column.xml)
  * [`admin-2columns-left.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Theme/view/adminhtml/page_layout/admin-2columns-left.xml)
  * [`admin-empty.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Theme/view/adminhtml/page_layout/admin-empty.xml)
  * [`admin-login.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Theme/view/adminhtml/page_layout/admin-login.xml)

`empty.xml` является базовым page layout для фронта и все маджентоские page layouts его расширяют, сам лаяут добавляется в корневой контейнер `root`, в который добавляются все остальные контейнеры.

## Блоки и темйлетый

Блоки в M2 схожи с блоками в M1. 

Добавление блока:
```xml
<referenceBlock name="top.links">
  <block class="Magento\Customer\Block\Account\AuthorizationLink" name="authorization-link"
         template="account/link/authorization.phtml"/>
</referenceBlock>
```

Блок [`Magento\Framework\View\Element\Template`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/Template.php) и его наследники имеют темлейты. Они хранятся в phtml файлах, и могут находиться в (в поряде fallback'а):

* _`<theme_dir>`/`<Namespace_Module>`/templates_
* _`<parent_theme_dir>`/`<Namespace_Module>`/templates_
* _`<module_dir>`/view/frontend/templates_
* _`<module_dir>`/view/base/templates_

Способы названчить блоку темплейт:

* Параметр блока `template` в лаяуте:
```xml
<block class="Magento\Catalog\Block\Product\View" name="product.info" template="Magento_Catalog::product/view/form.phtml" />
```
так-же данный параметр можно поменять через `<referenceBlock>`:
```xml
<referenceBlock name="breadcrumbs" template="Magento_Catalog::product/breadcrumbs.phtml">
```
* инструкцией `<action>` можно взывать метод `setTemplate`, но данная инструкция объявлена утаревшей.
* можно передать параметер `template` в конструктор блока в конструкции `<arguments>`.
* в самом клаасе можно просетать свойство `$_template`.

Пути для темлейта можно указать:

* С указанием модуля:
```xml
<block class="Magento\Catalog\Block\Product\View" name="product.info" template="Magento_Catalog::product/view/form.phtml" />
```
темлейт будет искаться в контексте указанного модуля
* Без указания: 
```xml
<block class="Magento\Tax\Block\Sales\Order\Tax" name="tax" template="order/tax.phtml"/>
```
темплейт будет искаться в контексте текущего модуля.

### Вызов методов и связь с блоком

В темлейте для доступа к блоку используется переменная `$block`, есть также возможность использовать `$this`, как в M1, однако такое поведение считается устаревшим, у `$this` есть метод `$this->helper('name')`, но его использоване в M2 счтается плохой практикой.

## Короткий тег 

Для вывода в шаблонах можно использовать короткий тег `<?= $block->getText() ?>`, аналогично `<?php echo $block->getText();  ?>`

## Передача параметров

В блоках все параметры переданные ему в `<arguments>` в лаяуте попадют в итоге в `$_data` поле блока, соответственно для них в темлейте и блоке можно использовать гетеры и сетеры.

## Экранирование

* [`escapeHtml()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L899) — экранирует html, вторым параметром принимает массив разрешённых тегов.
* [`escapeUrl()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L967) — экранирует URL, не позволяет использовать псевдо-протоколы внутри ссылок
* [`escapeJs()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L912) — используется для экранирования js переменных, строк и чисел. 
* [`escapeHtmlAttr()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L926) — для экранирования значения одного аттрибута html-тега:
```php
<input name="field" value="<?php echo $block->escapeHtmlAttr($block->getFieldValue()) ?>" />
```

## Методы вывода потомков

В M2 детей могут иметь контейнеры и блоки, контейнеры выводят их автоматиески, в блоках это происходит явно, с помощью методов:

* [`getChildHtml()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L511) — принимает имя или алиас дочернего блока или контейнера, возвращет вывод
* [`getBlockHtml()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L569) —  как `getChildHtml()` но выводит любые элементы на странице, например `formkey`.
* [`getChildChildHtml()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L541) — примнимает имя/алиас дочерненего и имя/алиас дочернего дочернего элемента, если второй параметр не указан вернёт вывод всех дочерних элементов элемента первого параметра.

## etc/view.xml

В блоках можно получать значения из `etc/view.xml`, для этого используется метод `$block->getVar(‘product_image_size’, ‘Magento_Wishlist’)`, вторым параметром задаётся модуль со значением, если не задан используется текущий модуль.

## View модели

[View models](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/view-models.html)

В M2 предлогается разделять бизнес логику и функционал рендеринга в блоках, т.е блоки должны использоваться только для рендеринга, а другая логика должна содержаться в другом классе, для этого предлагают использовать вью модели. 

Добавляется вью модель как аргумент блока:
```xml
<block name="test" template="test.phtml">
    <arguments>
        <argument name="viewModel" xsi:type="object">Oggetto\Training\ViewModel\Product</argument>
    </arguments>
</block>
```
Затем в блоке она доступна посредством `$block->getViewModel()` или `$block->getData(‘viewModel’)`.

Класс view модели должен реализоывать [`Magento\Framework\View\Element\Block\ArgumentInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/Block/ArgumentInterface.php)
