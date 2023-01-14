# Demonstrate ability to use quote, quote item, address, and shopping cart rules in checkout

>Describe how to modify these models and effectively use them in customizations.

>Describe how to customize the process of adding a product to the cart. Which different scenarios should you take into account?

[How to Customize the Checkout Process in Magento 2](https://belvg.com/blog/how-to-customize-the-checkout-process-in-magento-2.html)

Квота в Magento используется в качестве корзнины клиента.

## Основные модели

Функционал квоты находится в модуле [`Magento_Quote`](https://github.com/magento/magento2/tree/2.4/app/code/Magento/Quote), наиболее важные классы:

* [`Magento\Quote\Model\Quote`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote.php) — модель квоты, реализует [`Magento\Quote\Api\Data\CartInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Api/Data/CartInterface.php)
  * [`Magento\Quote\Model\Quote\Item`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item.php) — модель айтема квоты (продукта), реализует [`Magento\Quote\Api\Data\CartItemInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Api/Data/CartItemInterface.php) и наследует [`\Magento\Quote\Model\Quote\Item\AbstractItem`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item/AbstractItem.php)
  * [`Magento\Quote\Model\Quote\Address`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address.php) — модель адреса, реализует [`Magento\Quote\Api\Data\AddressInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Api/Data/AddressInterface.php)
    * [`Magento\Quote\Model\Quote\Address\Rate`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Rate.php) — модель тарифа доставки
    * [`Magento\Quote\Model\Quote\Address\Item`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Address/Item.php) — модель айтема, используется для мультишипинг доставки, наследует [`Magento\Quote\Model\Quote\Item\AbstractItem`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item/AbstractItem.php)
    * [`Magento\Quote\Model\Quote\Address\Total`](https://github.com/magento/magento2/tree/2.4/app/code/Magento/Quote/Model/Quote/Address/Total) — некоторые модели тоталов, про них будет подробно в следующей теме.
  * [`Magento\Quote\Model\Quote\Payment`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Payment.php) — модель оплаты, реализует [`Magento\Quote\Api\Data\PaymentInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Api/Data/PaymentInterface.php)

## Структура БД

В базе квоты использует следующие таблицы:

* `quote` — квоты
* `quote_address` — адреса квот
* `quote_address_item` — айтемы адресов квот (для мультишипинга)
* `quote_id_mask` — хеши для гостевых квот. [How to Get Quote Id by Masked id Magento 2](https://www.rakeshjesadiya.com/get-quote-id-by-masked-id-magento-2/)
* `quote_item` — айтемы квот
* `quote_item_option` — опции айтемов квот
* `quote_payment` — оплаты квот
* `quote_shipping_rate` — тарифы доставки квот

### Интересные поля в `quote`

* `ext_shipping_info` — текстовое поле, не используется маджентой
* `trigger_recollect` — флаг пересчёта корзины, он [вызывает пересчёт итогов](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote.php#L2444). Он проставляется когда изменяются цены или статус продукта меняется на "отключен".

## Describe how to modify these models and effectively use them in customizations.

### Валидация количества

1. Измнение количества в айтеме происходит в методе [`setQty()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item.php#L349)
2. В `setQty()` вызывается ивент `sales_quote_item_qty_set_after`
3. [`Magento\CatalogInventory\Observer\QuantityValidatorObserver::execute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogInventory/Observer/QuantityValidatorObserver.php#L34)
4. [`Magento\CatalogInventory\Model\Quote\Item\QuantityValidator::validate()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogInventory/Model/Quote/Item/QuantityValidator.php#L108)

### Добавление в корзину

Добавление продукта в контроллере [`Magento\Checkout\Controller\Cart\Add`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Controller/Cart/Add.php):

1. Для добавление используется [`Magento\Checkout\Model\Cart::addProduct()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/Cart.php#L367), этот класс считается устаревшим.
2. Вызывается [`Magento\Quote\Model\Quote::addProduct()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote.php#L1611).
3. У инстанса типа продукта вызывается [`prepareForCartAdvanced()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L450), который готовит продукт для корзины, у разных типов может быть своя реализация в методе [`_prepareProduct()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L366). `prepareForCartAdvanced()` возвращает продукты которые должны быть добавлены в квоту.
4. Создаётся новый айтем корзины или берётся существующий, если он уже есть для продукта.
5. Для айтема корзины вызывается [Magento\Quote\Model\Quote\Item\Processor::prepare()](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item/Processor.php#L89), он вызывает у айтема [`addQty()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item.php#L329) и сетает `price` у айтема и `custom_price`, если она присутвует в запросе.
6. В квоте тригерится событие `sales_quote_product_add_after` с добавленными айтемами
7. В Cart тригерится событие `checkout_cart_product_add_after` с резльтатом добавления и добавляемым продуктом.
8. ИД добавленного продукта добавляет в сессию чекаута: session->setLastAddedProductId
9. После добавления вызывается метод [`Magento\Checkout\Model\Cart::save()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/Cart.php#L574), который вызывает пересчёт тарифов доставок и тоталов квоты, так-же здесь тригерятся инвенты:
  * `checkout_cart_save_before`
  * `checkout_cart_save_after`
10. В контролере добавления в корзину [тригерится ивент `checkout_cart_add_product_complete`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Controller/Cart/Add.php#L124).

### Обновление корзины

Обновление корзины в контролере [`Magento\Checkout\Controller\Cart\UpdatePost`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Controller/Cart/UpdatePost.php):

1. Вызывается[`Magento\Checkout\Model\Cart::suggestItemsQty()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/Cart.php#L464), этот класс считается устаревшим. Метод обрабатывает переданный массив и возвращает валидное количество продуктов, которое можно добавить в корзину
2. Вызывается [`Magento\Checkout\Model\Cart::updateItems()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/Cart.php#L504) Метод обновляет айтемы корзины (удаляет или изменяет количество)
3. Вызывается [`Magento\Checkout\Model\Cart::save()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Checkout/Model/Cart.php#L574), который вызывает пересчёт тарифов доставок и тоталов квоты, так-же здесь тригерятся инвенты:
  * `checkout_cart_save_before`
  * `checkout_cart_save_after`

## Describe how to customize the process of adding a product to the cart.

* [`prepareForCartAdvanced`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L450) — используется для подготовки продуктов к добавлению в корзину, можно написать плагин на этот метод
* При подготовкве опций продукта [тригерится ивент `catalog_product_type_prepare_full_options`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L611)
* [Magento\Quote\Model\Quote\Item\Processor::prepare()](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item/Processor.php#L89) — используется для подготовки айтема, можно написать плагин и изменить добавляемое количество продукта или `custom_price`.
* Можно использовать ивенты вызываемые во время процесса добавления, такие как `sales_quote_product_add_after`, `checkout_cart_product_add_after`
* В файлах _`<module-dir>`/etc/catalog\_attributes.xml_ в ноде `<group name=”quote_item”>` содержатся атрибуты продукта которые добавляются в модель продукта в квоте:
```xml
<group name="quote_item">
    <attribute name="sku"/>
    <attribute name="type_id"/>
    <attribute name="name"/>
</group>
```

### Which different scenarios should you take into account?

* Добавление продукта из каталога в корзину;
* Добавление продукта из вишлиста в корзину;
* Добавление всех продуктов вишлиста в корзину;
* Создание заказа из админки/с фронта;
* Перезаказ заказа из админки/с фронта;
* Конфигурация добавление продукта — кастомные опции;
* Комбинация квот авторизированого клиента, если он добавил продукты как гость и у него уже имеется квота.

## Мердж квот

При авторизации пользователя с существующей корзиной запускается процесс мерджа квот, он происходит в методе [`Magento\Quote\Model\Quote::merge()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote.php#L2401). В нём сравнивается каждый айтем методом [`Magento\Quote\Model\Quote\Item::compare()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item.php#L516), в нём для сравнения используется [`Magento\Quote\Model\Quote\Item\Compare::compare()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item/Compare.php#L69). Если сравнение вернёт `false`, то будет добавлен новый айтем, иначе будет увеличено количество существующего в корзине айтема (для которого при сравнении вернулось `true`).

Айтемы сравниваются по:

* ИД продукта
* опциям (из метода [`Magento\Quote\Model\Quote\Item::getOptionsByCode()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/Quote/Item.php#L604))
