# Describe price functionality

>Identify the basic concepts of price generation in Magento. How would you identify what is composing the final price of the product? How can you customize the price calculation process?

>Describe how price is rendered in Magento. How would you render price in a given place on the page, and how would you modify how the price is rendered?

[Price Generation Functionality & Concepts in Magento 2](https://belvg.com/blog/the-functionality-and-basic-concepts-of-price-generation-in-magento-2.html)

## Identify the basic concepts of price generation in Magento

Работа с ценами в продукте происходит в классе [`Magento\Catalog\Model\Product\Type\Price`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/Price.php).

Каждый продукт в M2, имеет несколько цен.

Базовые типы цен, доступные для всех типов продуктов:

* base_price (принимает параметр количества) — минимальная цена из спец.цены или tire_price для переданного количества
* tier_price — цена продукта, зависимая от количества продуктов в корзине (вид скидки, настраивается в админке)
* final_price — финальная цена продукта, после примения всех условий (на уровне каталога)
* special_price — специальная цена товара, может иметь ограничение во времени примения
* price — значения для атрибута price из базы

### PriceInfo

Также для работы с ценами используется [`Magento\Framework\Pricing\PriceInfo\Base`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/PriceInfo/Base.php) реализующий [`Magento\Framework\Pricing\PriceInfoInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/PriceInfoInterface.php). Объект возвращающий `PriceInfoInterface` должен реализовывать [`Magento\Framework\Pricing\SaleableInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/SaleableInterface.php), его реализовывает модель продукта, она создаёт объект `PriceInfoInterface` c помощью [`Magento\Framework\Pricing\PriceInfo\Factory`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/PriceInfo/Factory.php).

`Magento\Framework\Pricing\PriceInfo\Base` при создании принимает в себя две коллекции:

* [`Magento\Framework\Pricing\Price\Collection`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Price/Collection.php) — коллекция цен, содержит в себе пул моделей типов цен доступных для типа продукта
* [`Magento\Framework\Pricing\Adjustment\Collection`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Adjustment/Collection.php) — коллекция дополнений к ценам, например, налоги, содержит в себе пул дополнений

Для каждого типа цены есть свой класс, который наследут [`Magento\Framework\Pricing\Price\AbstractPrice`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Price/AbstractPrice.php) реализующий [`Magento\Framework\Pricing\Amount\AmountInterface\PriceInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Price/PriceInterface.php)

### How would you identify what is composing the final price of the product?

Цена получается следующим образом:

`$product->getPrice(\Magento\Catalog\Pricing\Price\FinalPrice::PRICE_CODE)->getAmount()` для симпл продукта.

1. [`Magento\Framework\Pricing\Price\AbstractPrice->getAmount()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Price/AbstractPrice.php#L99)
```php
if (!isset($this->amount[$this->getValue()])) {
    $this->amount[$this->getValue()] = $this->calculator->getAmount($this->getValue(), $this->getProduct());
}
return $this->amount[$this->getValue()];
```
2. `getValue()` — это абстрактный метод `AbstractPrice`, соответственно у каждого типа цены он свой. Для [`final_price` реализация такая](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Pricing/Price/FinalPrice.php#L42):
```php
public function getValue()
{
    return max(0, $this->getBasePrice()->getValue());
}
```
3. `getValue()` у `final_price` вызывает [`Magento\Catalog\Pricing\Price\BasePrice->getValue()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Pricing/Price/BasePrice.php)
```php
if ($this->value === null) {
    $this->value = false;
    foreach ($this->priceInfo->getPrices() as $price) {
        if ($price instanceof BasePriceProviderInterface && $price->getVal() !== false) {
                $this->value = min($price->getValue(), $this->value !== false ? $this->value: $price->getValue());
            }
        }
    }
return $this->value;
```
в методе берутся все цены реализующие [`Magento\Framework\Pricing\Price\BasePriceProviderInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Price/BasePriceProviderInterface.php) и среди них берётся минимальное значение, для симпла это такие цены:
* [`regular_price`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Pricing/Price/RegularPrice.php) — цена из атрибута `price` продукта
* [`tier_price`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Pricing/Price/TierPrice.php) — tier цена (когда цена меняется в зависмости от количества продуктов в корзине)
* [`special_price`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Pricing/Price/SpecialPrice.php) — цена из атрибута `special_price` продукта
* [`catalog_rule_price`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Pricing/Price/CatalogRulePrice.php) — цена со скидкой по каталожным правилам
4. После получения значения `final_price`, это значение передаёт в метод `getAmount()` класса реализующего [`Magento\Framework\Pricing\Adjustment\CalculatorInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Adjustment/CalculatorInterface.php)
5. Для симпла (и остальных, кроме бандлов) таким классом является [`Magento\Framework\Pricing\Adjustment\Calculator`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Adjustment/Calculator.php)
6. `Calculator->getAmount()` проходится по всем дополнения (adjustment) к цене, если нужно добавлять это дополнение к цене то добавляет его. Для симпла используется 3 дополнения, это всё различные налоги:
* [`tax`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Pricing/Adjustment.php)
* [`weee`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Weee/Pricing/Adjustment.php)
* [`weee_tax`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Weee/Pricing/TaxAdjustment.php)
7. Поле подсчёта дополнений для финального итога создаётся объект реализующий [`Magento\Framework\Pricing\Amount\AmountInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Amount/AmountInterface.php), его реализует класс [`Magento\Framework\Pricing\Amount\Base`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Amount/Base.php)
8. `Base` при приведении к строке вызвает метод [`getValue()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Amount/Base.php#L59) без параметров, при необходимости в `getValue()` можно передать код дополнения чтобы исключть его из результата.
```php
if ($exclude === null) {
    return $this->amount;
} else {
    if (!is_array($exclude)) {
        $exclude = [(string)$exclude];
    }
    $amount = $this->amount;
    foreach ($exclude as $code) {
        if ($this->hasAdjustment($code)) {
            $amount -= $this->adjustmentAmounts[$code];
        }
    }
    return $amount;
}
```

Для остальных типов продуктов логика схожа, но у каждого могут быть свои особености:

* [`configurable`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/ConfigurableProduct/Pricing/Price/FinalPrice.php) — выбирает минимальную `final_price` среди своих дочерних продуктов
* [`grouped`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/GroupedProduct/Pricing/Price/FinalPrice.php) — выбирает минимальную `final_price` среди своих дочерних продуктов
* [`bundle`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Bundle/Pricing/Price/FinalPrice.php) — так-же как у симпла + [`Magento\Bundle\Pricing\Price\BundleOptionPrice->getValue()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Bundle/Pricing/Price/BundleOptionPrice.php)


### How can you customize the price calculation process?

Есть несколько способов:

1. Создать новый тип цены и добавить его в пул цен.
```xml
<virtualType name="MyVendor\MyModule\Pricing\Price\Pool" type="Magento\Framework\Pricing\Price\Pool">
  <arguments>
      <argument name="prices" xsi:type="array">
          <item name="my_price" xsi:type="string">MyVendor\MyModule\Pricing\Price\MyPrice</item>
      </argument>
      <argument name="target" xsi:type="object">Magento\Catalog\Pricing\Price\Pool</argument>
  </arguments>
</virtualType>
```
2. Созадть плагин для нужного типа цены для метода `getValue()`.
3. (Для конкретного типа продуктов) Перезаписать класс предоставляющий пул цен, например для `bundle` это сделано так, в `di.xml`:
```xml
<type name="Magento\Framework\Pricing\PriceInfo\Factory">
    <arguments>
        <argument name="types" xsi:type="array">
            <item name="bundle" xsi:type="array">
                <item name="infoClass" xsi:type="string">Magento\Bundle\Pricing\PriceInfo</item>
                <item name="prices" xsi:type="string">Magento\Bundle\Pricing\Price\Collection</item>
            </item>
        </argument>
    </arguments>
</type>
    <virtualType name="Magento\Bundle\Pricing\Price\Collection" type="Magento\Framework\Pricing\Price\Collection">
    <arguments>
        <argument name="pool" xsi:type="object">Magento\Bundle\Pricing\Price\Pool</argument>
    </arguments>
</virtualType>
    <virtualType name="Magento\Bundle\Pricing\Price\Pool" type="Magento\Framework\Pricing\Price\Pool">
    <arguments>
        <argument name="prices" xsi:type="array">
            <item name="regular_price" xsi:type="string">Magento\Bundle\Pricing\Price\BundleRegularPrice</item>
            <item name="final_price" xsi:type="string">Magento\Bundle\Pricing\Price\FinalPrice</item>
            <item name="tier_price" xsi:type="string">Magento\Bundle\Pricing\Price\TierPrice</item>
            <item name="special_price" xsi:type="string">Magento\Bundle\Pricing\Price\SpecialPrice</item>
            <item name="custom_option_price" xsi:type="string">Magento\Catalog\Pricing\Price\CustomOptionPrice</item>
            <item name="base_price" xsi:type="string">Magento\Catalog\Pricing\Price\BasePrice</item>
            <item name="configured_price" xsi:type="string">Magento\Bundle\Pricing\Price\ConfiguredPrice</item>
            <item name="configured_regular_price" xsi:type="string">Magento\Bundle\Pricing\Price\ConfiguredRegularPrice</item>
            <item name="bundle_option" xsi:type="string">Magento\Bundle\Pricing\Price\BundleOptionPrice</item>
            <item name="bundle_option_regular_price" xsi:type="string">Magento\Bundle\Pricing\Price\BundleOptionRegularPrice</item>
            <item name="catalog_rule_price" xsi:type="string">Magento\CatalogRule\Pricing\Price\CatalogRulePrice</item>
        </argument>
    </arguments>
</virtualType>
```
4. Полность перезаписать класс типа цены используя `preference` в `di.xml`

## Describe how price is rendered in Magento.

Для отображения цены используется блок [`Magento\Catalog\Pricing\Render\FinalPriceBox`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Pricing/Render/FinalPriceBox.php) и темплейт `Magento_Catalog::product/price/final_price.phtml`

Для рендеринга используется метод [`renderAmount`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Render/PriceBox.php#L137)
```php
$arguments = array_replace($this->getData(), $arguments);
//@TODO AmountInterface does not contain toHtml() method
return $this->getAmountRender($amount, $arguments)->toHtml();
```

1. `getAmountRender()` возвращает объект реализующий [`Magento\Framework\Pricing\Render\AmountRenderInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Render/AmountRenderInterface.php) его реализует класс [`Magento\Framework\Pricing\Render\Amount`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Render/Amount.php), 
2. У него вызывается `_toHtml()`:
```php
$adjustmentRenders = $this->getAdjustmentRenders();
if ($adjustmentRenders) {
    $adjustmentHtml = $this->getAdjustments($adjustmentRenders);
    if (!$this->hasSkipAdjustments() ||
        ($this->hasSkipAdjustments() && $this->getSkipAdjustments() == false)) {
            $this->adjustmentsHtml = $adjustmentHtml;
    }
}
$html = parent::_toHtml();
return $html;
```
3. Для Amount по умолчанию используется шаблон `Magento_Catalog::product/price/amount/default.phtml`
4. Там-же редерятся дополнения для цены, для них используются классы реализующие [`Magento\Framework\Pricing\Render\AdjustmentRenderInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Render/AdjustmentRenderInterface.php) его реализует класс [`Magento\Framework\Pricing\Render\AbstractAdjustment`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Render/AbstractAdjustment.php) все рендеры дополений наследуются от него. 

### How would you render price in a given place on the page, and how would you modify how the price is rendered?

Для отображения блока с ценой нужно добавить блок `Magento\Catalog\Pricing\Render` в лаяуте, передав ему нужные параметры:
```xml
<block class="Magento\Catalog\Pricing\Render" name="product.price.myprice">
  <arguments>
      <argument name="price_render" xsi:type="string">product.price.render.default</argument>
      <argument name="price_type_code" xsi:type="string">final_price</argument>
      <argument name="zone" xsi:type="string">item_view</argument>
  </arguments>
</block>
```

Для измения темплейтов цен можно создать собственный хендл `catalog_product_prices.xml`, в нём можно изменить блок `render.product.prices`:
```xml
<layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/layout_generic.xsd">
  <referenceBlock name="render.product.prices">
      <arguments>
          <argument name="myprice" xsi:type="array">
              <item name="prices" xsi:type="array">
                  <item name="final_price" xsi:type="array">
                      <item name="render_class" xsi:type="string">MyVendor\MyModule\Pricing\Render\FinalPriceBox</item>
                      <item name="render_template" xsi:type="string">MyVendor_MyModule::product/price/final_price.phtml</item>
                  </item>
              </item>
          </argument>
      </arguments>
  </referenceBlock>
</layout>
```

## Индекс

Ценовой индекс собирает минимальные цены из всех источников (price, special price, tier price, catalog price rule) и хранит их в `catalog_product_index_price`.

Модель для индекса указывается в `indexerModel` типа продукта. Модели реализуют интерфейс [`Magento\Framework\Indexer\DimensionalIndexerInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Indexer/DimensionalIndexerInterface.php).
