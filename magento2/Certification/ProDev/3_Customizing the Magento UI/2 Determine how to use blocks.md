# Determine how to use blocks

>Demonstrate an understanding of block architecture and its use in development. Which objects are accessible from the block? What is the typical block’s role?

>Identify the stages in the lifecycle of a block. In what cases would you put your code in the `_prepareLayout()`, `_beforeToHtml()`, and `_toHtml()` methods? How would you use events fired in the abstract block?

>Describe how blocks are rendered and cached.

>Identify the uses of different types of blocks. When would you use non-template block types? In what situation should you use a template block or other block types?

[How to Use Blocks in Magento 2 Development](https://belvg.com/blog/how-to-use-blocks-in-magento-2-development.html)

Блоки используются для рендеринга.

Все блоки с темлейтами наследуются от [`Magento\Framework\View\Element\Template`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/Template.php) по дефолту в блоки также инджектится [`Magento\Framework\View\Element\Template\Context`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/Template/Context.php)

## Жизненый цикл

Существует две фазы жизненного цикла блока:

* Генерация
* Рендеринг

Блоки создаются в момент создания лаяута, в этот момент строится структура блоков, задаются дочерние блоки, у блоков вызвается метод `prepareLayout()`. При этом ничего не рендерится.

### Генерация

1. [`Magento\Framework\View\Page\Config::publicBuild()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Page/Config.php#L235)
2. [`Magento\Framework\View\Page\Config::build()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Page/Config.php#L221)
3. [`Magento\Framework\View\Layout\Builder::build()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout/Builder.php#L59)
4. [`Magento\Framework\View\Layout\Builder::generateLayoutBlocks()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout/Builder.php#L116)
5. [`Magento\Framework\View\Layout::generateElements()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout.php#L323)
6. [`Magento\Framework\View\Layout\GeneratorPool::process()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout/GeneratorPool.php#L77)

GeneratorPool проходит через все генераторы и генерирует все запланированные элементы.

### Рендеринг 

1. [`Magento\Framework\View\Result\Page::render()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Result/Page.php#L240)
2. [`Magento\Framework\View\Layout::getOutput()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout.php#L958)

В этот момент уже имеется весь xml лаяют для сгенирированной страницы и созданы все классы блоков.

3. [`Magento\Framework\View\Layout::renderElement()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout.php#L492)
4. [`Magento\Framework\View\Layout::renderNonCachedElement()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Layout.php#L535)

В последнем методе проверяется тип элмента, для которого происходит рендер, для блоков вызывается [`toHtml()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L665)

## Кастомизация

Изменить генерацию и рендеринг блока можно в этих методах:

* `_prepareLayout()`
  * добавить [дочерние блоки](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Block/Adminhtml/Attribute/Edit/Options/AbstractOptions.php#L24)
  * добавить [табы(для админки)](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/Block/System/Design/Edit/Tabs.php#L26)
  * [изменить тайтл](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Wishlist/Block/Customer/Sharing.php#L82)
  * [добавить педжер, бредкрабсы и т.п.](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/Block/Order/History.php#L123)
  * [изменить рендер](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Block/Adminhtml/Form.php#L23)
* `_beforeToHtml()`
  * [подготовка данных](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Block/Product/ProductList/Related.php#L118)
  * [назначение значений](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/Block/Widget/Form/Element.php#L72)
  * [добавление дочерних боков](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Block/Adminhtml/Create/Items.php#L88)
*  `_toHtml()` — здесь можно непосредственно изменять процесс рендеринга блока

## События

* в `toHtml()` генерируется [`view_block_abstract_to_html_before`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L667), он генерируется перед рендерингом, в него передаётся блок.
* в `toHtml()` генерируется [`view_block_abstract_to_html_after`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L685), он генерируется после рендеринга, в него передаётся блок и объект содержащий сгенирированный вывод блока, который можно модифицировать.

## Кеширование

Вывод блоков кешируется. При рендеринге блока сначала происходит попытка загрузить его из кеша в методе [`_loadCache()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L1103), если блока там нет, то происходит его рендеринг и вывод сохраняется в кеш в методе [`_saveCache()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L1146)

Для идентификации в кеше использкется `cache_key`, получаемый методом [`getCacheKey()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L1040). При сохранении используется `cache_tags` из метода [`getCacheTags()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L1064). Для определения времени жизни кеша используется [`getCacheLifetime()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php#L1084), значение указывается в секундах, если нужно отключить кеш можно указать 0 или не устаниваливать значение, по умолчанию все блоки не кешируются.
Стоит учитывать, что если обнулить время жизни кеша, то кеширование не будет действовать только для данного типа блоков, при этом блок может кешироваться полностраничным кешированием, чтобы это избежать нужно отключать кеширование для блока в лаяуте:
```xml
<block class="Magento\Paypal\Block\Payflow\Link\Iframe" template="payflowlink/redirect.phtml" cacheable="false"/>
```
При неличии некешированного блока маджента отключает полностраничный кеш для страницы.

## Типы блоков

* [`Magento\Framework\View\Element\AbstractBlock`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/AbstractBlock.php) – абстрактный блок
* [`Magento\Framework\View\Element\Template`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/Template.php) – блок с темплейтом
* [`Magento\Framework\View\Element\Text`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/Text.php) – рендерит просто текст
* [`Magento\Framework\View\Element\FormKey`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/FormKey.php) – рендерит скрытый инпут с formKey
* [`Magento\Framework\View\Element\Messages`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/Messages.php) – рендерит уведомления

## Email

[Email templates](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/templates/template-email.html)

Темплейты для email подерживают такой же фелбек как и обычные темплейты блоков.

Темлейты распологаются в:
* _view/`<area>`/email_

### Инструкции

* `{{config path="general/store_information/hours"}}` — значение конфига
* `{{var subscriber_data.confirmation_link}}` или `{{var store_phone}}` — переменная
* `{{inlinecss file="css/email-inline.css"}}` — добавление css как инлайнового стиля
* `{{css file="css/email.css"}}` — добавление css
* `{{trans "Text"}}` — перевод

Ветление:
```
{{if logo_width}}
    width="{{var logo_width}}"
{{else}}
    width="200"
{{/if}}
```

[Magento 2 email template template directives](https://gordonlesti.com/magento-2-email-template-template-directives/)