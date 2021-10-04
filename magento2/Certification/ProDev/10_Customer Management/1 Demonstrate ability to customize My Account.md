# Demonstrate ability to customize My Account

>Describe how to customize the “My Account” section. How do you add a menu item? How would you customize the “Order History” page?

## Describe how to customize the “My Account” section.

### How do you add a menu item?

Для добавление нового айтема в меню аккаунта нужно добавить новый блок класса [`Magento\Customer\Block\Account\SortLinkInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Block/Account/SortLinkInterface.php) к блоку `customer_account_navigation`. Для этого:

1. Создать файл `customer_account.xml`
* для темы: _Magento\_Customer/layout/customer\_account.xml_
* для модуля: _view/frontend/layout/customer\_account.xml_ 

2. Добавить в него:
```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceBlock name="customer_account_navigation">
            <block class="Magento\Customer\Block\Account\SortLinkInterface" name="customer-account-navigation-my-link">
                <arguments>
                    <argument name="path" xsi:type="string">mylink</argument>
                    <argument name="label" xsi:type="string" translate="true">My link</argument>
                    <argument name="sortOrder" xsi:type="number">600</argument>
                </arguments>
            </block>
        </referenceBlock>
    </body>
</page>
```

[`Magento\Customer\Block\Account\SortLinkInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Block/Account/SortLinkInterface.php) реализует класс [`Magento\Customer\Block\Account\SortLink`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Block/Account/SortLink.php), он наследуется от [`Magento\Framework\View\Element\Html\Link\Current`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/Html/Link/Current.php) который наследуется от блока с темплейтом. Если темплейт для блока не задан то html генерируется в методе [`toHtml()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Element/Html/Link/Current.php#L106)

### How would you customize the “Order History” page?

Лаяует для раздела: [`sales_order_history.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/view/frontend/layout/sales_order_history.xml)
Основной блок: [`Magento\Sales\Block\Order\History`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/Block/Order/History.php)

Базовый лаяут добавляет два контейнера:

* sales.order.history.extra.column.header
* sales.order.history.extra.container

В дефолтном темлейте [`view/frontend/templates/order/history.phtml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/view/frontend/templates/order/history.phtml) эти контейнеры выводятся:

* sales.order.history.extra.column.header — в хедерах таблицы заказов полсе поля "Date"
* sales.order.history.extra.container — в строке таблицы заказа после столбца "Date"

Таким обзраом если необходимо добавить новые колонки в таблицу можно добавить новый блоки в эти контейнеры:
```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <!-- This will add additional column header to order list -->
        <referenceBlock name="sales.order.history.extra.column.header">
            <block class="Magento\Framework\View\Element\Template" name="your.additional.column.header" template="Namespace_Module::columnheader.phtml"/>
        </referenceBlock>
        <!-- You can access current order using $this->getOrder() inside the template "-->
        <referenceBlock name="sales.order.history.extra.container">
            <block class="Magento\Framework\View\Element\Template" name="your.additional.column.data" template="Namespace_Module::columndata.phtml"/>
        </referenceBlock>
    </body>
</page>
```
