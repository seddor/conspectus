# Demonstrate ability to use totals models

>Describe how to modify the price calculation process in the shopping cart. How can you add a custom totals model or modify existing totals models?

Тоталы используются для подсчёта итогов, итоги считаются для адресов затем они записываются в квоту. Кроме случаев, когда используется мультишипинг, тоталы считаются для двух адресов: доставки (shipping) и оплаты (billing), для виратульаных корзин (с виртуальными айтемами) только для адреса оплаты.

Тоталы наследуются от класса [`Magento\Quote\Model\Quote\Address\Total\AbstractTotal`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Total/AbstractTotal.php), он реализует [`Magento\Quote\Model\Quote\Address\Total\CollectorInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Total/CollectorInterface.php), [`Magento\Quote\Model\Quote\Address\Total\ReaderInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Total/ReaderInterface.php).

Подсчёт тоталов вызывается в методе [`Magento\Quote\Model\Quote::collectTotals()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote.php#L1993):
```php
if ($this->getTotalsCollectedFlag()) {
    return $this;
}

$total = $this->totalsCollector->collect($this);
$this->addData($total->getData());

$this->setTotalsCollectedFlag(true);
return $this;
```

`totalsCollector` это [`Magento\Quote\Model\Quote\TotalsCollector`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/TotalsCollector.php), у него вызывается [`collect()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/TotalsCollector.php#L125), в нём обнуляются основные итоги квоты, происходит перерасчёт тоталов для адресов квоты, происходит валидация `grand_total` и `base_grand_total` квоты и генерируются ивенты:
* `sales_quote_collect_totals_before` — до подсчёта, передаётся квота
* `sales_quote_collect_totals_after` — после подсчёта, передаётся квота

Расчёт тоталов для адреса происходит в [`Magento\Quote\Model\Quote\TotalsCollector::collectAddressTotals()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/TotalsCollector.php#L247), генерируются ивенты:
* `sales_quote_address_collect_totals_before`
* `sales_quote_address_collect_totals_after`

Сбором тоталов занимается класс [`Magento\Quote\Model\Quote\Address\Total\Collector`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Total/Collector.php)

Обычно итоги имеют по два значения с приставкой `base_` (цена в валюте вебсайта) и без неё (цена в валюте текущего стор вью).

## Describe how to modify the price calculation process in the shopping cart. 

### How can you add a custom totals model or modify existing totals models?

#### Создане тотала

1. В файле _`<module-dir>`/etc/sales.xml_ регистируются новые тоталы:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Sales:etc/sales.xsd">
    <section name="quote">
        <group name="totals">
            <item name="custom_total" instance="Vendor\Module\Model\Totals\Custom" sort_order="500"/>
        </group>
    </section>
</config>
```
2. Создать класс для тотала, он должен наследоваться от [`Magento\Quote\Model\Quote\Address\Total\AbstractTotal`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Total/AbstractTotal.php):
```php
Namespace Vendor\Module\Model\Total;
 
class Custom extends \Magento\Quote\Model\Quote\Address\Total\AbstractTotal
{
    /**
     * Custom constructor.
     */
    public function __construct()
    {
        $this->setCode('custom_total');
    }
 
    /**
     * @param \Magento\Quote\Model\Quote $quote
     * @param \Magento\Quote\Api\Data\ShippingAssignmentInterface $shippingAssignment
     * @param \Magento\Quote\Model\Quote\Address\Total $total
     * @return $this
     */
    public function collect(
        \Magento\Quote\Model\Quote $quote,
        \Magento\Quote\Api\Data\ShippingAssignmentInterface $shippingAssignment,
        \Magento\Quote\Model\Quote\Address\Total $total
    ) {
        parent::collect($quote, $shippingAssignment, $total);
 
        $items = $shippingAssignment->getItems();
        if (!count($items)) {
            return $this;
        }
 
        //we will add an additional amount of 150 to the order as an example
        $amount = 150;
 
        $total->setTotalAmount('custom_total', $amount);
        $total->setBaseTotalAmount('custom_total', $amount);
        $total->setCustomAmount($amount);
        $total->setBaseCustomAmount($amount);
        $total->setGrandTotal($total->getGrandTotal() + $amount);
        $total->setBaseGrandTotal($total->getBaseGrandTotal() + $amount);
 
        return $this;
    }
 
    /**
     * @param \Magento\Quote\Model\Quote\Address\Total $total
     */
    protected function clearValues(\Magento\Quote\Model\Quote\Address\Total  $total)
    {
        $total->setTotalAmount('subtotal', 0);
        $total->setBaseTotalAmount('subtotal', 0);
        $total->setTotalAmount('tax', 0);
        $total->setBaseTotalAmount('tax', 0);
        $total->setTotalAmount('discount_tax_compensation', 0);
        $total->setBaseTotalAmount('discount_tax_compensation', 0);
        $total->setTotalAmount('shipping_discount_tax_compensation', 0);
        $total->setBaseTotalAmount('shipping_discount_tax_compensation', 0);
        $total->setSubtotalInclTax(0);
        $total->setBaseSubtotalInclTax(0);
    }
 
    /**
     * @param \Magento\Quote\Model\Quote $quote
     * @param \Magento\Quote\Model\Quote\Address\Total $total
     * @return array
     */
    public function fetch(
        \Magento\Quote\Model\Quote  $quote,
         \Magento\Quote\Model\Quote\Address\Total $total
    ) {
        return [
            'code' => $this->getCode(),
            'title' => 'Custom Total',
            'value' => 150
        ];
    }
 
    /**
     * @return \Magento\Framework\Phrase
     */
    public function getLabel()
    {
        return __('Custom Total');
    }
}
```

Резултаты подсчётов записываются и передаются в инстанс класса [`Magento\Quote\Model\Quote\Address\Total`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Total.php), он является агументом методов тоталов `collect()` и `fecth()`, при необходимости изменить какой-нибудь тотал можно написать плагин на метод `collect()` и поменять значение для аргумента `$total`.

## Отображение тоталов

Тоталы рендарятся в корзине в UI-компонентах.

JS тоталы расширяют [`Magento_Checkout/js/view/summary/abstract-total`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/view/frontend/web/js/view/summary/abstract-total.js) и реализуют:

* getPureValue - чистый вывод значения
* getValue - форматированное, через [`Magento_Catalog/js/price-utils`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/view/base/web/js/price-utils.js) значение. Формат берётся из `window.checkoutConfig.priceFormat` для цен стора, и `window.checkoutConfig.basePriceFormat` для цен сайта.

Значения получаются из [`quote.getTotals()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/view/frontend/web/js/model/quote.js#L88), а там из `window.checkoutConfig.totalsData`. Данные в `window.checkoutConfig` предоставляет класс [`Magento\Checkout\Model\CompositeConfigProvider`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/CompositeConfigProvider.php) реализующий [`Magento\Checkout\Model\ConfigProviderInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/ConfigProviderInterface.php), он использует массив провайдеров реализующих [`Magento\Checkout\Model\ConfigProviderInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/ConfigProviderInterface.php). Дефолтная реализация находится в классе [`Magento\Checkout\Model\DefaultConfigProvider`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/DefaultConfigProvider.php)

В методе [`getTotalsData()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/DefaultConfigProvider.php#L632) используется [`Magento\Quote\Model\Cart\CartTotalRepository`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Cart/CartTotalRepository.php) реализующий [`Magento\Quote\Api\CartTotalRepositoryInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Api/CartTotalRepositoryInterface.php). У него вызывается метод [`get()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Cart/CartTotalRepository.php#L88), в котором берётся адрес (для виртальной квоты платёжный, иначе доставки) у адреса вызывается [`getTotals()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address.php#L1109), там вызывается метод [`Magento\Quote\Model\Quote\TotalsReader::fetch()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/TotalsReader.php#L40) там у тоталов вызывается метод `fecth()` и для тоталов собирается вывод.

## Тоталы для других моделей

Тоталы также есть для инвойсов (счетов) заказов и кредит мемо (возвратов средств) заказов.

Они также регистрируются в файле _`<module-dir>`/etc/sales.xml_:
```xml
<section name="order_invoice">
    <group name="totals">
        <item name="subtotal" instance="Magento\Sales\Model\Order\Invoice\Total\Subtotal" sort_order="50"/>
    </group>
</section>
<section name="order_creditmemo">
    <group name="totals">
        <item name="subtotal" instance="Magento\Sales\Model\Order\Creditmemo\Total\Subtotal" sort_order="50"/>
    </group>
</section>
```

Абстрактные классы тоталов:
* [`Magento\Sales\Model\Order\Invoice\Total\AbstractTotal`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/Model/Order/Invoice/Total/AbstractTotal.php) — для инвойсов
* [`Magento\Sales\Model\Order\Creditmemo\Total\AbstractTotal`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/Model/Order/Creditmemo/Total/AbstractTotal.php) — для кердит мемо
