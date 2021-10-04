# Demonstrate ability to use products and product types

>Identify/describe standard product types (simple, configurable, bundled, etc.). How would you obtain a product of a specific type? What tools (in general) does a product type model provide? What additional functionality is available for each of the different product types?

## Типы продуктов

[Using products and standard product types](https://belvg.com/blog/using-products-and-standard-product-types-simple-configurable-bundled-etc-in-magento-2.html)

Все типы продуктов наследуются от [Magento\Catalog\Model\Product\Type\AbstractType](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php)

Типы продуктов:

* [Simple](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/Simple.php)
* [Configurable](https://github.com/magento/magento2/blob/2.4/app/code/Magento/ConfigurableProduct/Model/Product/Type/Configurable.php)
* [Grouped](https://github.com/magento/magento2/blob/2.4/app/code/Magento/GroupedProduct/Model/Product/Type/Grouped.php)
* [Virtual](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/Virtual.php)
* [Bundle](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Bundle/Model/Product/Type.php)
* [Downloadable](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Downloadable/Model/Product/Type.php)
* Gift card (EE)

Основные различия:

* могут ли иметь сток
* могут ли быть контейнерами для других типов товаров

### Создание типа товара

#### 1. Сконфигураровать новый тип в `etc/product_types.xml`
```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Catalog:etc/product_types.xsd">
    <type name="bundle" label="Bundle Product" modelInstance="Magento\Bundle\Model\Product\Type" composite='true' indexPriority="40" sortOrder="50">
        <priceModel instance="Magento\Bundle\Model\Product\Price" />
        <indexerModel instance="Magento\Bundle\Model\ResourceModel\Indexer\Price" />
        <stockIndexerModel instance="Magento\Bundle\Model\ResourceModel\Indexer\Stock" />
        <allowedSelectionTypes>
            <type name="simple" />
            <type name="virtual" />
        </allowedSelectionTypes>
        <customAttributes>
            <attribute name="refundable" value="true"/>
        </customAttributes>
    </type>
</config>
```

* `composite` — определяет, может ли товар такого типа включать в себя другие;
* `allowedSelectionTypes` — определяет, какие типы товаров может в себя включать товар этого типа, но насколько вижу, это используется только в бандл товарах;
* `priceModel` — отвечает за расчет цены на пдп и других местах;
* `indexerModel` — определяет класс, который индексит цены, они используются, например, на странице категории и результатах поиска;
* `customAttributes` — позволяет задавать различные другие настройки, например `taxable` и `refundable`.

Можно еще добавить `composableTypes` - определяет, какие товары каких типов могут содержаться в других товарах.

Например:
```xml
<composableTypes>
    <type name="simple" />
    <type name="virtual" />
</composableTypes>
```

#### 2. Если товар можно будет покупать, добавить его в `etc/sales.xml`
```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Sales:etc/sales.xsd">
    <order>
        <available_product_type name="bundle"/>
    </order>
</config>
```

#### 3. Добавить новый тип для атрибутов

Добавить новый тип в колонку apply_to в таблице catalog_eav_attribute для атрибутов, которые должны быть у этого нового типа. Если apply_to пустая, то значит что атрибут применяется ко всем типам товаров. 
Если не пустая, значит, только к определенным.

#### 4. Настроить price_renderer в `view/base/layout/catalog_product_prices.xml`
```xml
<?xml version="1.0"?>
<layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/layout_generic.xsd">
    <referenceBlock name="render.product.prices">
        <arguments>
            <argument name="bundle" xsi:type="array">
                <item name="prices" xsi:type="array">
                    <item name="tier_price" xsi:type="array">
                        <item name="render_template" xsi:type="string">Magento_Bundle::product/price/tier_prices.phtml</item>
                    </item>
                    <item name="final_price" xsi:type="array">
                        <item name="render_class" xsi:type="string">Magento\Bundle\Pricing\Render\FinalPriceBox</item>
                        <item name="render_template" xsi:type="string">Magento_Bundle::product/price/final_price.phtml</item>
                    </item>
                    <item name="bundle_option" xsi:type="array">
                        <item name="amount_render_template" xsi:type="string">Magento_Bundle::product/price/selection/amount.phtml</item>
                    </item>
                </item>
            </argument>
        </arguments>
    </referenceBlock>
</layout>
```

#### 5. Добавить рендерер для корзины `view/base/layout/catalog_product_prices.xml`
```xml
<?xml version="1.0"?>
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceBlock name="checkout.cart.item.renderers">
            <block class="Magento\Bundle\Block\Checkout\Cart\Item\Renderer" name="checkout.cart.item.renderers.bundle" as="bundle" template="Magento_Checkout::cart/item/default.phtml">
                <block class="Magento\Checkout\Block\Cart\Item\Renderer\Actions" name="checkout.cart.item.renderers.bundle.actions" as="actions">
                    <block class="Magento\Checkout\Block\Cart\Item\Renderer\Actions\Edit" name="checkout.cart.item.renderers.bundle.actions.edit" template="Magento_Checkout::cart/item/renderer/actions/edit.phtml"/>
                    <block class="Magento\Checkout\Block\Cart\Item\Renderer\Actions\Remove" name="checkout.cart.item.renderers.bundle.actions.remove" template="Magento_Checkout::cart/item/renderer/actions/remove.phtml"/>
                </block>
            </block>
        </referenceBlock>
    </body>
</page>
```

#### 6. Добавить остальные рендереры (см. vendor/magento/module-bundle/view/[AREA]/layout).

В админке это: айтемы заказа, инвойса, шипмента и кредитмемо.
На фронте в основном: для писем и печати заказа, инвойса, шипмента и кредитмемо, для айтемов чекаута и т.д.

---

### Как выбрать тип товара

#### Физический товар идущий в комплейкте с подпиской
Физический товар - симпл, подписка - виртуальный, а вместе это можно офрмить в бандл товар. Также можно закастомить, чтобы убрать возможность выбирать количество, например, чтобы подписка всегда шла только одна.

#### Видеокурс
Или виртуальный товар, или downloadable, зависит от нужд.

#### Очки и рецепт к ним
Простой товар с custom опцией, в которую можно загрузить файл.

---


## How would you obtain a product of a specific type?

Тип продукта хранится в `catalog_product_entity.type_id`. Соотвественно получить продукты определённого типа можно отфильтровав коллекцию по этому полю.

## What tools (in general) does a product type model provide?

Получил инстанс модели типа можно методом [`Product::getTypeInstance()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product.php#L682).

С помощью модели типа можно сделать следующее:

* Работа с дочерними продуктами, если они есть для типа
  * [`getChildrenIds()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L249) — у `grouped`, `bundle`, `configurable`
  * [`getAssociatedProducts()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L1099) — только у `grouped`, у `configurable` есть свой метод [`getUsedProducts()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/ConfigurableProduct/Model/Product/Type/Configurable.php#L1287)
  * [`setImageFromChildProduct()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L1077) — реализован только у `configurable`, ставит родительскому товару первую доступную картинку из дочерних товаров
*  Получить информацию о родительских продуктах, если они есть для типа
  * [`getParentIdsByChild()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L261)
* работа с атрибутами:
  * [`getSetAttributes()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L272) — получить массив атрибутов продуктового атрибут сета
  * [`getAttributeById()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L316)
массив атрибутов продуктового атрибут сета
* получение свойств товара, часто изменяется в типах:
  * [`isVirtual()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L335)
    * по умолчанию — `false`
    * `virtual` — `true`
    * `bundle` — `true`, если все выбранные товары виртуальные
    * `configurable` — если продукт из опции `simple_product` виртуальный, то `true`
    * `gift card` — если выбран виртуальный тип карты
  * [`isSalable()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L346)
    * по умолчанию — если статус продукта включаен и у продукта `is_salable` = `true`, то `true`
    * `bundle` — если среди выбраных есть те у которых `isSalable` и при этом, те, что не доступны не являются обязательной опцией, то `true`, если все доступны для продажи у родительского продукта сетается поле `all_items_salable`
    * `configurable` — если родительский товар включен и все его дочерние в стоке и `isSalable` у них `true`, то `true`
    * `downloadable` — если продукт включен, и у него есть ссылка, то `true`
    * `gift card` — если продукт включен, и у него есть доступные опции для выбора номинала, то `true`
  * [`isComposite()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L753) — по умолчанию `false`, `true` у `grouped`, `bundle`, `configurable`
  * [`canConfigure()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L765) — по умолчанию `false`, `true` у `grouped`, `bundle`, `configurable`, `gift card`
  * [`canUseQtyDecimals()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L775) — по умолчанию `true`, `false` у `gift card`
  * [`hasWeight()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L1065) — по умолчанию `true`, `false` у `downloadable`, `grouped`, `virtual`
* Подготовка продукта для добавления в корзину:
  * [`_prepareProduct()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L366) — так-же методы [`processConfiguration()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L431), [`prepareForCartAdvanced()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L450), [`prepareForCart()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L467) которые внутри себя вызывают `_prepareProduct()`. Подробнее подготовка продуктов к корзине будет рассмотрена ниже.
* методы для работы с опциями товара:
  * [`getSpecifyOptionMessage()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L562)
  * [`_prepareOptions()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L576) — метод готовит опции для корзины
  * [`checkProductBuyState()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L625) — проверяет текущее состояние выбора опций продукта, бросает исключение, если что-то не так
  * [`getOrderOptions()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L652) — возвращщает опции для заказа
  * [`hasOptions()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L850)
  * [`hasRequiredOptions()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L880)

### Подготовка продукта к добавлению в корзину 

При добавлении продукта в корзину происходит его трансформация в айтем квтоты.

Добавление происходит в методе [`Magento\Quote\Model\Quote::addProduct()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote.php#L1611)

В этом методе вызывается метод модели типа [`prepareForCartAdvanced()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L450), он вызвает метод [`_prepareProduct()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L366). реализация `_prepareProduct` отличается для разных типов:

* `simple` — готовит опции продукта, возвращает один продукт, который сохраняется в айтем квтоты
* `configurable` — возвращает массив дочерних продуктов, добавляет к ним опции с информацией о родительском продкте и конфигурируемых опциях
* `grouped`  — возвращает дочерние продукты, добавлят к ним опции с информацией о родителе
* `bundle` — возвращает дочерние продукты
* `downloadable` — так-же как `simple` + генерирует ссылку.

## Связанные продукты

Связи продуктов хранятся в таблице `catalog_product_link`. Типы продуктов в `catalog_product_link_type`, там доступны такие типы:

* [relation](https://docs.magento.com/user-guide/catalog/settings-advanced-related-products.html) — это дополненя для текущего продукта
* super — связи групповых товаров
* [up_sell](https://docs.magento.com/user-guide/catalog/settings-advanced-up-sells.html) — это продукты которые дороже и лучше текущего.
* [cross_sell](https://docs.magento.com/user-guide/catalog/settings-advanced-cross-sells.html) — продукты расчитанные на импульсивные покупки, обычно располагаются на чекауте (по сути это как жвачка в супермаркете). 

Атрибуты для связей хранятся в `catalog_product_link_attribute`, например, это может быть позиция или количество товара (для групповых). Значения атрибутов хранится в `catalog_product_link_attribute_<type>`, тип может быть:
* decimal
* int
* varchar

Для получения связанных продуктов есть такие методы:

* [`Magento\Catalog\Model\ResourceModel\Product\Link::getChildrenIds()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/ResourceModel/Product/Link.php#L250) — возвращает ИД связанных товаров нужного типа
* [`Magento\Catalog\Model\Product::getUpSellProductCollection()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product.php#L1395)
* [`Magento\Catalog\Model\Product::getCrossSellProductCollection()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product.php#L1445)
* [`Magento\Catalog\Model\Product::getRelatedProductCollection()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product.php#L1321)
* [`Magento\GroupedProduct\Model\Product\Type\Grouped::getAssociatedProducts()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/GroupedProduct/Model/Product/Type/Grouped.php#L203) — товары связаные с групповым товаром
