# Demonstrate ability to customize shipping and payment methods

>Describe shipping methods architecture. How can you create a new shipping method? What is the relationship between carriers and rates?

>Describe how to troubleshoot shipping methods and rate results. Where do shipping rates come from? How can you debug the wrong shipping rate being returned?

>Describe how to troubleshoot payment methods. What types of payment methods exist? What are the different payment flows?

## Describe shipping methods architecture

Методы доставки настраиваются в _Stores — Confuguration — Sales — Delivery methods_.

Каждый метод доставки так-же является курьером, у одного курьера может быть несколько тарифов доставки (shipping rate), именно тарифы доставки отображаются на чекауте и их выбирает пользователь.

### Методы доставки на чекауте

Получение методов:

1. Запрос на 
  * /V1/carts/:cartId/shipping-methods
  * /V1/carts/:cartId/estimate-shipping-methods
  * /V1/carts/:cartId/estimate-shipping-methods-by-address-id
2. Все три метода в конечном итоге приходят к методу [`Magento\Quote\Model\ShippingMethodManagement::getShippingMethods()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/ShippingMethodManagement.php#L315) в этом методе происходит вызов перерасчёта тоталов и тарифов доставки (shipping rate).
3. Полученные тарифы доставки забираются методом [`Magento\Quote\Model\Quote\Address::getGroupedAllShippingRates()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address.php#L890), этот метод возвращает массив тарифов сгруппированных по курьерским службам (методам доставки), затем этот массив приводится в плоский вид, модели конвертируются в дата-объекты и это отправляется в ответ.

### How can you create a new shipping method?

Наиболее простой пример курьерки [`Magento\OfflineShipping\Model\Carrier\Freeshipping`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/OfflineShipping/Model/Carrier/Freeshipping.php).

шаги создание курьерки:
1. Создать дефолтную конфигурацию в _`<module_dir>`/etc/config.xml_ в ноде `default/carriers/<carrier_name>`:
```xml
<default>
    <carriers>
        <flatrate>
            <active>1</active>
            <sallowspecific>0</sallowspecific>
            <model>Magento\OfflineShipping\Model\Carrier\Flatrate</model>
            <name>Fixed</name>
            <price>5.00</price>
            <title>Flat Rate</title>
            <type>I</type>
            <specificerrmsg>This shipping method is not available. To use this shipping method, please contact us.</specificerrmsg>
            <handling_type>F</handling_type>
        </flatrate>
    </carriers>
</default>
```
2. Создать раздел в системной конфигурации в _`<module_dir>`/etc/adminhtml/system.xml_:
```xml
<system>
    <section id="carriers" type="text" sortOrder="320" showInDefault="1" showInWebsite="1" showInStore="1">
        <group id="flatrate" translate="label" type="text" sortOrder="0" showInDefault="1" showInWebsite="1" showInStore="1">
            <label>Flat Rate</label>
            <field id="active" translate="label" type="select" sortOrder="1" showInDefault="1" showInWebsite="1" canRestore="1">
                <label>Enabled</label>
                <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
            </field>
            <field id="name" translate="label" type="text" sortOrder="3" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
                <label>Method Name</label>
            </field>
            <field id="price" translate="label" type="text" sortOrder="5" showInDefault="1" showInWebsite="1" canRestore="1">
                <label>Price</label>
                <validate>validate-number validate-zero-or-greater</validate>
            </field>
        </group>
    </section>
</system>
```
3. Сзодать модель курьерки, модель должна наследоваться от [`Magento\Shipping\Model\Carrier\AbstractCarrier`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Carrier/AbstractCarrier.php)
4. При необходимости можно добавить фронтовую валидация для курьерки в `checkout_cart_index.xml`, например как [здесь](https://github.com/magento/magento2/blob/2.4/app/code/Magento/OfflineShipping/view/frontend/layout/checkout_cart_index.xml)

### What is the relationship between carriers and rates?

Курьер может содержать в себе несколько тарифов.

## Describe how to troubleshoot shipping methods and rate results. 

### Where do shipping rates come from?

1. Для вызова подсчёта тарифов используется метод [`Magento\Quote\Model\Quot\Address::collectShippingRates()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address.php#L988), подсчёт тарифов доставки происходит при `getCollectShippingRates() == true`. Вызывать метод можно напрямую или он вызовется сам при пересчёте тоталов, в тотале [`shipping`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Total/Shipping.php#L84).
2. В методе `collectShippingRates()`, `getCollectShippingRates` меняется на `false`, удаляются прошлые шипинг рейты для адреса и происходит вызов  [`Magento\Quote\Model\Quot\Address::requestShippingRates()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address.php#L1020)
3. В методе `requestShippingRates()` создаётся инстанс [`Magento\Quote\Model\Quote\Address\RateRequest`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/RateRequest.php) и передаётся в [`Magento\Quote\Model\Quote\Address\RateCollectorInterface::collectRates()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/RateCollectorInterface.php#L18)
4. `RateCollectorInterface` реализует [`Magento\Shipping\Model\Shipping`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Shipping.php)  у него в методе [`collectRates()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Shipping.php#L204) из конфигурации достаются все коды курьерок и для каждого вызывается [`collectCarrierRates()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Shipping.php#L308)
5. В `collectCarrierRates()` по коду достаётся курьерка в методе [`Magento\Shipping\Model\Shipping::prepareCarrier()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Shipping.php#L274), там проверяется включена ли курьерка и если включена, то по коду создаётся модель курьерки, она наследует класс [`Magento\Shipping\Model\Carrier\AbstractCarrier`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Carrier/AbstractCarrier.php), который реализует [`Magento\Shipping\Model\Carrier\AbstractCarrierInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Carrier/AbstractCarrierInterface.php), затем курьерка валидируется и если всё ок, то возвращается в качестве результата.
6. После получения модели курьерки в `collectCarrierRates()` у курьерки вызывается метод `collectRates()`
7. `collectRates()` для каждой курьерки реазлизуется по своему, в результате он возвращает [`Magento\Shipping\Model\Rate\Result`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Rate/Result.php) или [`Magento\Quote\Model\Quote\Address\RateResult\Error`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/RateResult/Error.php) в случае ошибок, так-же можнет вернуться `false` когда у курьерки нет тарифов для текущего запроса.
8. В курьерка получившиеся тарифы упаковываются в [`Magento\Quote\Model\Quote\Address\RateResult\Method`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/RateResult/Method.php). Каждый тариф содержит примерно такие данные:
```
carrier = "flatrate"
carrier_title = "Flat Rate"
method = "flatrate"
method_title = "Fixed"
price = 5.0
cost = 5.0
```
9. Получившиеся результаты добавляются в [`Magento\Shipping\Model\Rate\CarrierResult`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Rate/CarrierResult.php)
10. Получившееся реультат возвращается в метод [`Magento\Quote\Model\Quot\Address::requestShippingRates()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address.php#L1069) и из этих резултатов создаются инстансы моделей тарифа [`Magento\Quote\Model\Quote\Address\Rate`]() и они добавляются в адрес в методе [`Magento\Quote\Model\Quot\Address::addShippingRate()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address.php#L975)

Затем тарифы доставки можно получить методом [`Magento\Quote\Model\Quot\Address::getGroupedAllShippingRates()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address.php#L890).

### How can you debug the wrong shipping rate being returned?

* В [`collectRates()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Shipping.php#L204) происходит сбор трифов.
* Курьерки валидируются в [`Magento\Shipping\Model\Shipping::prepareCarrier()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Shipping/Model/Shipping.php#L274)
* Сбор тарифов для каждой курьерки происходит в методе `collectRates()` модели курьерки.

## Describe how to troubleshoot payment methods

[Гайд Magento по интеграции методов оплаты](https://devdocs.magento.com/guides/v2.4/payments-integrations/bk-payments-integrations.html)

### Получение доступных методов оплаты

1. [`Magento\Quote\Model\PaymentMethodManagement::getList()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/PaymentMethodManagement.php#L107)
2. [`Magento\Payment\Model\MethodList::getAvailableMethods()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Payment/Model/MethodList.php#L68)
3. [`Magento\Payment\Model\PaymentMethodList::getActiveList()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Payment/Model/PaymentMethodList.php#L81) реализует [`Magento\Payment\Api\PaymentMethodListInterface`]() возращает массив включённых [`Magento\Payment\Api\Data\PaymentMethodInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Payment/Api/Data/PaymentMethodInterface.php)
4. [`Magento\Payment\Model\MethodList::getAvailableMethods()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Payment/Model/MethodList.php#L73) для каждого метода создаётся инстанс [`Magento\Payment\Model\MethodInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Payment/Model/MethodInterface.php) проверяется для квоты в методах `MethodInterface::isAvailable()` и [`Magento\Payment\Model\MethodList::_canUseMethod()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Payment/Model/MethodList.php#L90)
5. Полученные методы отправляются в ответ.

### Платёжные команды

Платёжные команды — это методы метода платежа, которые выполняют определённое действие над платежом. Список команд из [`Magento\Payment\Model\Method\Adapter`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Payment/Model/Method/Adapter.php):
* fetch_transaction_information
* order
* authorize — зарегистрировать транзакцию в сервисе оплаты. Там она завалидируется и вернет статус
* capture — выполнить авторизованную транзакцию  т. е. списать деньги
* refund — возврат денег (после capture)
* cancel
* void — отмена авторизации транзакции
* acceptPayment
* denyPayment

Для добавление команды нужно:
1. Найти нужный тип пула команд для нужного метода оплаты
2. Добавить команду в аргумент `commands`:
```xml
<virtualType name="AmazonCommandPool" type="Magento\Payment\Gateway\Command\CommandPool">
    <arguments>
        <argument name="commands" xsi:type="array">
            <item name="authorize" xsi:type="string">AmazonAuthorizeCommand</item>
            <item name="capture" xsi:type="string">AmazonCaptureStrategyCommand</item>
            <item name="sale" xsi:type="string">AmazonSaleCommand</item>
            <item name="settlement" xsi:type="string">AmazonSettlementCommand</item>
            <item name="void" xsi:type="string">AmazonVoidCommand</item>
            <item name="cancel" xsi:type="string">AmazonVoidCommand</item>
            <item name="refund" xsi:type="string">AmazonRefundCommand</item>
        </argument>
    </arguments>
</virtualType>
```
3. Создать виртуальный тип для [`Magento\Payment\Gateway\Command\GatewayCommand`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Payment/Gateway/Command/GatewayCommand.php):
```xml
<virtualType name="AmazonAuthorizeCommand" type="Amazon\Payment\Gateway\Command\AmazonAuthCommand">
    <arguments>
        <argument name="requestBuilder" xsi:type="object">Amazon\Payment\Gateway\Request\AuthorizationRequestBuilder</argument>
        <argument name="handler" xsi:type="object">Amazon\Payment\Gateway\Response\CompleteAuthHandler</argument>
        <argument name="transferFactory" xsi:type="object">Amazon\Payment\Gateway\Http\TransferFactory</argument>
        <argument name="validator" xsi:type="object">AmazonAuthorizationValidators</argument>
        <argument name="client" xsi:type="object">Amazon\Payment\Gateway\Http\Client\AuthorizeClient</argument>
        <argument name="errorMessageMapper" xsi:type="object">Amazon\Payment\Gateway\ErrorMapper\VirtualErrorMessageMapper</argument>
    </arguments>
</virtualType>
```

### What types of payment methods exist? What are the different payment flows?

Методы оплаты делятся на три категории:

* Geteway — когда платёжная информация передаётся в Magento (например карта), или карта токенизируется на фронте и authorization/capture проихсодит в Magento.
* offline — когда оплата не подразумевает подключения к стороним сервисам, напримен Check/Money Order, Банковский перевод, оплата при получении, бесплатный заказ. Все стандартные методы оплаты этого типа находятся в модуле [`Magento_OfflinePayments`](https://github.com/magento/magento2/tree/2.4/app/code/Magento/OfflinePayments)
* Hosted — когда для оплаты пользователя перенаправляют на веб-сайт платёжной системы где и происходит оплата.
