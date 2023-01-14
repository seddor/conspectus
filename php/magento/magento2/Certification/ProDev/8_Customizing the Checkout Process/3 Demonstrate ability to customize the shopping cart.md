# Demonstrate ability to customize the shopping cart

>Describe how to implement shopping cart rules. What is the difference between sales rules and catalog rules? How do sales rules affect performance? What are the limitations of the native sales rules engine?

>Describe add-to-cart logic in different scenarios. What is the difference in adding a product to the cart from the product page, from the wishlist, by clicking Reorder, and during quotes merge?

>Describe the difference in behavior of different product types in the shopping cart. How are configurable and bundle products rendered? How can you create a custom shopping cart renderer?

>Describe the available shopping cart operations. Which operations are available to the customer on the cart page? How can you customize cart edit functionality? How would you create an extension that deletes one item if another was deleted? How do you add a field to the shipping address?

## Describe how to implement shopping cart rules. 

За корзиночные правила отвечает модуль [`Magento_SalesRule`](https://github.com/magento/magento2/tree/2.4/app/code/Magento/SalesRule).

## Структура БД

Структура корзиночных правил в БД состоит из следующих таблиц:

Основные таблицы:
* `salesrule` — основная информация о правилах
* `salesrule_label` — назнавия для разных сторов
* `salesrule_product_attribute` — содержит связь правил и, используемых в условиях (применения правила (Conditions) и применения к продуктам (Actions)), продуктовых атрибутов. Используется в [`плагине`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Plugin/QuoteConfigProductAttributes.php#L37) к [`Magento\Quote\Model\Quote\Config::getProductAttributes()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Config.php#L26), добавляя в ответ атрибуты из таблицы.
* `salesrule_website` — соотношения правил и сайтов
* `salesrule_customer_group` — соотношения правил и групп кастомеров
* `salesrule_coupon` — купоны правил
Трекинг использования:
* `salesrule_coupon_usage` — использование купонов
* `salesrule_customer` — использование правил кастомерами
Таблицы для отчётов:
* `salesrule_coupon_aggregated`
* `salesrule_coupon_aggregated_updated`
* `salesrule_coupon_aggregated_order`

## Применение правил

Правила применяются с помощью тоталов, для правил используются такие тоталы:

* [discount (`Magento\SalesRule\Model\Quote\Discount`)](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Quote/Discount.php)
* [shipping_discount (`Magento\SalesRule\Model\Quote\Address\Total\ShippingDiscount`)](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Quote/Address/Total/ShippingDiscount.php)

### discount

1. [`Magento\SalesRule\Model\Quote\Discount::collect()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Quote/Discount.php#L106)
2. Для валидации используется [`Magento\SalesRule\Model\Validator`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Validator.php). Применение происходит на уровне айтемов квоты. Сначла айтемы валидируются, айтемы с `getNoDiscount() == true` и не прошедшие валидацю в методе [`Magento\SalesRule\Model\Validator::canApplyDiscount()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Validator.php#L571), и дочерние айтемы пропускаются. Для остальных применяется [`Magento\SalesRule\Model\Validator::process()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Validator.php#L263). Перед применением генерируется событие [`sales_quote_address_discount_item`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Quote/Discount.php#L162)

Валидаторы в `canApplyDiscount()` берутся из пула [`Magento\SalesRule\Model\Validator\Pool`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Validator/Pool.php), в `canApplyDiscount()` используются все валидаторы с типом `discount`, добавить валидаторы можно в `di.xml`:
```xml
<type name="Magento\SalesRule\Model\Validator\Pool">
    <arguments>
        <argument name="validators" xsi:type="array">
            <item name="discount" xsi:type="array">
                <item name="giftcard_validator" xsi:type="object">Magento\GiftCard\Model\Validator\Discount</item>
            </item>
        </argument>
    </arguments>
</type>
```
Валидаторы должны имплементировать `Zend_Validate_Interface`
3. В `process()` у айтема обнуляются поля с корзиночными скидками и применяется [`Magento\SalesRule\Model\RulesApplier::applyRules()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/RulesApplier.php#L106). В нём айтем ещё раз валидируются, для валидных айтемов вызывается [`Magento\SalesRule\Model\RulesApplier::applyRule()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/RulesApplier.php#L187)
4. В `applyRule()` вызывается [`Magento\SalesRule\Model\RulesApplier::getDiscountData()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/RulesApplier.php#L220), в котором используется [`CalculatorFaMagento\SalesRule\Model\Rule\Action\Discount\CalculatorFactory`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Rule/Action/Discount/CalculatorFactory.php), фабрика возвращает определённый класс для каждого типа скидок. Классы скидок наследуются от класса [`Magento\SalesRule\Model\Rule\Action\Discount\AbstractDiscount`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Rule/Action/Discount/AbstractDiscount.php) он реализует [`Magento\SalesRule\Model\Rule\Action\Discount\DiscountInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Rule/Action/Discount/DiscountInterface.php). Класс скидки подсчитывает величину скидки и возвращает в ответ инстанс класса [`Magento\SalesRule\Model\Rule\Action\Discount\Data`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Rule/Action/Discount/Data.php) содержащего данные по скидкам, затем вызывается ивент `salesrule_validator_process`. Данные из объекта с результатами записываются в айтем корзины. 

### shipping_discount

1. [`Magento\SalesRule\Model\Quote\Address\Total\ShippingDiscount::collect()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Quote/Address/Total/ShippingDiscount.php) если в адресе нулевая стоимость доставки то дальнейшее применение скидки на доставку не происходит
2. Применение скидки на доставку происходит в [`Magento\SalesRule\Model\Validator::processShippingAmount()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/SalesRule/Model/Validator.php#L299), там валидируются правила и применяется скидка.

### Бесплатная доставка

Бесплатнае доставка применяется в тотале [shipping (`Magento\Quote\Model\Quote\Address\Total\Shipping`)](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Total/Shipping.php).

1. В [`collect()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Total/Shipping.php#L53) тотала используется [`Magento\Quote\Model\Quote\Address\FreeShippingInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/FreeShippingInterface.php)
2. Интерфейс реализует класс [`Magento\OfflineShipping\Model\Quote\Address\FreeShipping`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/OfflineShipping/Model/Quote/Address/FreeShipping.php) в нём вызывается [`isFreeShipping()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/OfflineShipping/Model/Quote/Address/FreeShipping.php#L43), в котором проверяются айтемы квоты, если у айтема `getNoDiscount() == true` или он дочерний, то он пропускается, для остальных вызывается [`Magento\OfflineShipping\Model\SalesRule\ExtendedCalculator::processFreeShipping()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/OfflineShipping/Model/SalesRule/Calculator.php#L30)
3. В `processFreeShipping()` проверяются правила и если в каком-то есть бесплатная доставка и она применима для корзины, то бесплатная доставка применяется.

### What is the difference between sales rules and catalog rules?

* Каталожные правила применяются до корзины, т.е. скидка применяется к `final_price` продукта, она сразу видна в каталоге и в корзину продукт попадает уже со скидкой.
* Корзиночные правила применяются в корзине, в процессе подсчёта тоталов
* Корзиночные правила могут применятся по купонам
* Корзиночные правила могут влиять на цену доставки
* Для условий корзиночного правила используются не только атрибуты товара, но и атрибуты корзины

### How do sales rules affect performance?

Подсчёт правил может влиять на производительность подсчёта итогов корзины, особено при большом колчестве правил без купонов, т.к. они применяются автоматически и проверяются при каждом пересчёте итогов. Так-же могут быть проблемы с производительностью при большом количестве купонов в базе.

### What are the limitations of the native sales rules engine?

* Можно применить только один купон.
* Правло не может добавлять другие товары в корзину, только предоставлять скидку на находящиеся в корзине.

## Describe add-to-cart logic in different scenarios. 

### What is the difference in adding a product to the cart from the product page, from the wishlist, by clicking Reorder, and during quotes merge?

#### Product page

Контроллер: [`Magento\Checkout\Controller\Cart\Add`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Controller/Cart/Add.php)
Ивенты:
  * `checkout_cart_add_product_complete`
  * `checkout_cart_product_add_after`
  * `sales_quote_product_add_after`
  * `sales_quote_add_item`
Для добавление использует [`Magento\Checkout\Model\Cart::addProduct()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/Cart.php#L367)

#### Wishlist

Контроллер: [`Magento\Wishlist\Controller\Index\Cart`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Wishlist/Controller/Index/Cart.php)
Ивенты:
  * `checkout_cart_product_add_after`
  * `sales_quote_product_add_after`
  * `sales_quote_add_item`
Для добавление использует [`Magento\Wishlist\Model\Item::addToCart()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Wishlist/Model/Item.php#L405) который вызывает [`Magento\Checkout\Model\Cart::addProduct()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/Cart.php#L367)

#### Reorder

Контроллер: [`Magento\Sales\Controller\AbstractController\Reorder`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/Controller/AbstractController/Reorder.php)
Ивенты:
  * `checkout_cart_product_add_after`
  * `sales_quote_product_add_after`
  * `sales_quote_add_item`
Для добавление использует [`Magento\Sales\Model\Reorder\Reorder::execute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/Model/Reorder/Reorder.php#L132) который вызывает [`Magento\Checkout\Model\Cart::addProduct()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/Cart.php#L367)

#### Quote merge

Происходит в методе: [`Magento\Quote\Model\Quote::merge()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote.php#L2385). В нём если для айтема козины найден такой-же айтем то вызывается [`Magento\Quote\Model\Quote\Item::merge()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item/Processor.php#L116) иначе добавляется новый айтем методом [`Magento\Quote\Model\Quote::addItem()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote.php#L1590)

Ивенты:
  * `sales_quote_merge_before`
  * `sales_quote_add_item`
  * `sales_quote_merge_after`


## Describe the difference in behavior of different product types in the shopping cart.

Для кждого типа продуктов можно использовать свой рендер. Рендеры регестируются в блоке [`Magento\Framework\View\Element\RendererList`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/RendererList.php). Рендеры добавляются в хендл `checkout_cart_item_renderers`, для блока `checkout.cart.item.renderers`.

Пример:
```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceBlock name="checkout.cart.item.renderers">
            <block class="Magento\ConfigurableProduct\Block\Cart\Item\Renderer\Configurable" as="configurable" template="Magento_Checkout::cart/item/default.phtml">
                <block class="Magento\Checkout\Block\Cart\Item\Renderer\Actions" name="checkout.cart.item.renderers.configurable.actions" as="actions">
                    <block class="Magento\Checkout\Block\Cart\Item\Renderer\Actions\Edit" name="checkout.cart.item.renderers.configurable.actions.edit" template="Magento_Checkout::cart/item/renderer/actions/edit.phtml"/>
                    <block class="Magento\Checkout\Block\Cart\Item\Renderer\Actions\Remove" name="checkout.cart.item.renderers.configurable.actions.remove" template="Magento_Checkout::cart/item/renderer/actions/remove.phtml"/>
                </block>
            </block>
        </referenceBlock>
    </body>
</page>
```
Все блоки рендеров наследуются от класса [`Magento\Checkout\Block\Cart\Item\Renderer`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Block/Cart/Item/Renderer.php) многие используют этот же класс для рендеринга. Кастомные рендеры есть у:

* `configurable` — [`Magento\ConfigurableProduct\Block\Cart\Item\Renderer\Configurable`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/ConfigurableProduct/Block/Cart/Item/Renderer/Configurable.php)
* `bundle` — [`Magento\Bundle\Block\Checkout\Cart\Item\Renderer`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Bundle/Block/Checkout/Cart/Item/Renderer.php)
* `downloadable` — [`Magento\Downloadable\Block\Checkout\Cart\Item\Renderer`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Downloadable/Block/Checkout/Cart/Item/Renderer.php)
* `grouped` — [`Magento\GroupedProduct\Block\Cart\Item\Renderer\Grouped`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/GroupedProduct/Block/Cart/Item/Renderer/Grouped.php)

### How are configurable and bundle products rendered?

* `configurable` — [`Magento\ConfigurableProduct\Block\Cart\Item\Renderer\Configurable`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/ConfigurableProduct/Block/Cart/Item/Renderer/Configurable.php)
* `bundle` — [`Magento\Bundle\Block\Checkout\Cart\Item\Renderer`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Bundle/Block/Checkout/Cart/Item/Renderer.php)

### How can you create a custom shopping cart renderer?

Нужно создать блок рендера и прописать его в хендле `checkout_cart_item_renderers`, для блока `checkout.cart.item.renderers`:

```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceBlock name="checkout.cart.item.renderers">
            <block class="\Vendor\Module\Block\Cart\CustomRenderer" as="custom-type" template="Magento_Checkout::cart/item/default.phtml">
                <block class="Magento\Checkout\Block\Cart\Item\Renderer\Actions" name="checkout.cart.item.renderers.custom.actions" as="actions">
                    <block class="Magento\Checkout\Block\Cart\Item\Renderer\Actions\Edit" name="checkout.cart.item.renderers.custom.actions.edit" template="Magento_Checkout::cart/item/renderer/actions/edit.phtml"/>
                    <block class="Magento\Checkout\Block\Cart\Item\Renderer\Actions\Remove" name="checkout.cart.item.renderers.custom.actions.remove" template="Magento_Checkout::cart/item/renderer/actions/remove.phtml"/>
                </block>
            </block>
        </referenceBlock>
    </body>
</page>
```

После этого продукты с типом `custom-type` будут использовать кастомный рендер.

## Describe the available shopping cart operations.

### Which operations are available to the customer on the cart page?

* Изменение количества продуктов
* Изменение опций продутов (для конфигурируемых и бандлов)
* Удаление продуктов из корзины
* Добавление продуктов в вишлист
* Добавление/удаление купона

### How can you customize cart edit functionality?

[Customize Checkout](https://devdocs.magento.com/guides/v2.4/howdoi/checkout/checkout_overview.html)
[How to customize a checkout step in Magento 2](https://www.mageplaza.com/devdocs/custom-checkout-component-magento-2.html)

Для чекаута в M2 используются UI компоненты.

### How would you create an extension that deletes one item if another was deleted?

Можно использовать ивент `sales_quote_remove_item`, в нём передаётся удаляемый айтем в случае его удаления.

### How do you add a field to the shipping address?

[Add a new field in address form](https://devdocs.magento.com/guides/v2.4/howdoi/checkout/checkout_new_field.html)

Для EE есть возможность добавлять кастомные атрибуты в админке.
