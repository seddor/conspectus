# Ответы на вопросы с [MAGENTO 2 PRACTICE QUESTIONS](https://m2professional.dev/)

## 1

How to get original arguments in After Plugin?

* **Generated code pass it automatically, since from 3rd argument**
* It is impossible
* Use Before plugin, store arguments in Registry, call registry in After Plugin
* Use Around Plugin instead

[`Plugins`](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/plugins.html)

```php
public function afterLogin(\Magento\Backend\Model\Auth$authModel, $result, $username)
{
    $this->logger->debug('User ' . $username . ' signed in.');
}
```

## 2

Keeping maintainability in mind, how do you customize the format of addresses in emails?

*  Set the new template in etc/order_addresses.xml.
* Create a plugin for the Magento\Sales\Order\Address::getFormatted method.
* Use di.xml to add a new address formatter to Magento\Sales\Model\Order\Address\FormatterList.
* **Stores > Configuration > Customers > Customer Configuration > Address Templates.**

В настройке можно задавать форматирование адреса для:

* Текста
* Текста в одну строку
* HTML
* PDF

Затем получить отформатированные педставления можно в методе [`Magento\Customer\Model\Address\Config::getFormats()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/Address/Config.php#L116) возвращает массив дата-объектов. Определённый обхекст можно получить в методе [`Magento\Customer\Model\Address\Config::getFormatByCode()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/Address/Config.php#L183).

## 3

What is the relationship between block and template (choose 3)

* many block instances could be accessible in one template
* many templates could be assigned to one block instances
* one block could have multiple templates
* **many instances of one class could have many different templates**
* **many block instances could have one template**
* **many instances of one class could have the same template**

## 4

How to use a Content Delivery Network for Media Files?

* Replace url by plugin for getMediaUrl
* **Configure via Configuration/General/Web/Base URLs/ Base URL for User Media Files**
* Configure CDN vie di.xml CdnLists
* Create preference for MediaGallery class

Все урлы задаются в настройках по пути _Sotres — Configuration — General — Web — Base URLs_ и _Sotres — Configuration — General — Web — Base URLs (secure)_

Там можно задать:

* Base URL — базовый URL сайта
* Base link URL — для ссылок
* Bese static URL — для всего из _pub/static_
* Base media URL — для всего из _pub/media_

## 5

You added a grouped product to the cart. You see simple thumbnails image in cart. However want to see the same image for each option.
How to change it?

* Create mixin for cart item render and change it on frontend.
* Change item render via layout.xml.
* **Change config to Sales -> Checkout -> Shopping Cart -> Grouped Product Image -> Parent Product Thumbnail.**
* Create plugin aroundGetThumbnail for cart item block.

Там же можно выбрать картинку для **configurable** (из родителя или продукта).

## 6

What file in the etc/ folder do you need to create to instantiate an extension attribute?

* di.xml
* extension_attribute.xml
* **extension_attributes.xml**
* db/extension_attributes.xml

## 7

You are trying to figure out why some products have disappeared. One step in this process would be to rerun the index process. What is the command for this?

* bin/magento start:reindex
* **bin/magento indexer:reindex**
* bin/magento reindex:indexer
* bin/magento index:restart

[Manage the indexers](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-index.html)

## 8

How templates are initiated?

* It configured in app/design/frontend//
* Templates are initiated in layout files, and each layout block has an associated template
* Magento 2 gets they automatically from specific directories and render on specific page declared in config.xml

[Layout overview](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/layout-overview.html)

## 9

To create a new configuration type(Choos 4):

* **Create your XML file**
* Create plugin for Magento\Framework\Config\Reader\Filesystem afterRead and process you config
* **Create your XSD file.**
* **Define your configuration object in your di.xml**
* Define you config in config.xml
* **Define a reader by extending Magento\Framework\Config\Reader\Filesystem**

[Create or extend configuration types](https://devdocs.magento.com/guides/v2.4/config-guide/config/config-create.html)

## 10

You are facing a bug, which is supposedly caused by the customization of Magento\Catalog\Api\Product\RepositoryInterface::save().

To resolve the issue, you decide to find all logic which customizes this method.
Which two places do you search for customization declarations? (Choose 2)

* ***/di.xml**
* */plugins.xml
* */config.xml
* ***/events.xml**

* di.xml — может содержать плагины для этого класса
* events.xml — может отлавливать события готорые генерируются во время сохранения

## 11

You need to add a fee (additional to tax and shipping) to every order placed.

In what file do you specify the new total?

* etc/totals.xml
* etc/extension_attributes.xml
* etc/di.xml
* **etc/sales.xml**

В файле _`<module-dir>`/etc/sales.xml_ регистируются новые тоталы:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Sales:etc/sales.xsd">
    <section name="quote">
        <group name="totals">
            <item name="custom_total" instance="Vendor\Module\Model\Totals\Custom" sort_order="500"/>
        </group>
    </section>
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
</config>
```

## 12

The existing page layout does not meet your requirements. You created a new page layout 3-columns-double-footer.
Should you do something else to get your page in the layouts list?

* Create view/frontend/layout/3-columns-double-footer.xml, with
* Create datapatch and add 3-columns-double-footer to the DB table entity_layouts
* **Create view/frontend/layouts.xml with a node**
* Magneto will get it automatically because you created it in page_layout/3-columns-double-footer.xml

В _view/`<area>`/layouts.xml_ задаются доступные page layout:
```xml
<page_layouts xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/PageLayout/etc/layouts.xsd">
    <layout id="1column">
        <label translate="true">1 column</label>
    </layout>
    <layout id="2columns-left">
        <label translate="true">2 columns with left bar</label>
    </layout>
    <layout id="2columns-right">
        <label translate="true">2 columns with right bar</label>
    </layout>
    <layout id="3columns">
        <label translate="true">3 columns</label>
    </layout>
</page_layouts>
```

## 13

You need to make some modifications to an entity before it is saved to the database.
Your tool of choice is a plugin.
What are requirements for a plugin? (choose 3)

* The plugin class must not inherit the targeted class
* **The plugin class must be specified in di.xml**
* **The plugin method must begin with the type of the plugin**
* **The targeted method or class must not be marked final**
* The plugin class should not be marked abstract.

[`Plugins`](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/plugins.html)

## 14

Which entity types support an EAV functionality?(choose 3)

* CMS Block
* Order
* **Customer address**
* **Product**
* **Category**

Сейчас EAV есть для:
* Product
* Category
* Customer
* Customer address
Order, Invoice, Creditmemo, Shipment хоть и содержатся в `eav_entity_type`, на данный момент является обычными моделями.

## 15

There is plugin where it a product final price is needed. You expects the $product to be an instance of ProductInterface. But It doesn't have method getFinalPrice.
MagentoCatalog module's class realizes that interface and it has a method getFinalPrice().

* Change plugin to be dependent on MagentoCatalogModelProduct and MagentoCatalogModelRecourceModelProduct.
* Call getFinalPrice(), because MagentoCatalogModelProduct is a preference for ProductInterface by default.
* **Add:if (!$product instanceof MagentoCatalogModelProduct) throw new Exception(...)else return $product->getFinalPrice().**

## 16

Preferable way customizing Magento_Checkout/js/view/shipping in Magento checkout

* **Create mixing**
* Rewrite Magento_Checkout/js/view/shipping in theme
* Change files in vendor direcotry
* Add plugin for MagentoCheckoutModelConfigProviderInterface::getConfig

## 17

What kind of Catalog Input Type for Store Owner must you set to make the attributes filterable in Layered Navigation? (Choose 2)

* Image
* **Multiple Select**
* Text
* **Dropdown**

## 18

Keeping maintainability in mind, how do you add a new link to the customer account sidebar?

* Create customer_account_index.xml and add a block to the sidebar container
* Create a plugin for the customer Navigation class.
* Create customer_account_index.xml and add the link to the uiComponent configuration in jsLayout
* **Create customer_account.xml and add a block to the customer_account_navigation**

`customer_account.xml`:
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

## 19

In Magento. when is order closed?

* **When all items are refunded**
* When all products are invoiced
* When admin sets close
* When all items are shipped

When all items are shipped — complete
When all products are invoiced — processing

## 20

you are going to create console command that will use configs but your websites has many stores. How will you resolve config geting:

* inject StoreConfigInterface and use WEBSITE_SCOPE
* inject StoreConfig class and collect all configs
* inject StoreConfig class and use function getConfigByPath
* **inject StoreConfigInterface and use STORE_SCOPE**

## 21

What are valid plugin types:

* **after**
* instead
* post
* **around**
* **before**

[`Plugins`](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/plugins.html)

## 22

You inherited a class from 3rd party module.
Which type of dependency you should set in composer.json?

* Suggest
* **Require**

[Module dependencies: Hard dependencies](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_depend.html#hard-dependencies)

## 23

How to add a custom shipping carrier?(choose 2)

* **Create the carrier model that extends AbstractCarrier and implemets CarrierInterface**
* Create Action CollectRates that implement RateCalculationInterface and assign it to new carrier via DI.xml
* Add you carrier to white list vie di.xml
* **Create default config in "config.xml" file**

[Add custom shipping carrier](https://devdocs.magento.com/guides/v2.4/howdoi/checkout/checkout-add-custom-carrier.html)

## 24

You are customizing a third-party module and need to prevent an event handler from triggering.
Keeping upgradeability in mind, how do you do this?

* **Create an observer in events.xml with the same name and specify the disabled="true" attribute**
* Create a plugin that will disable the 3rd-party's event observer method
* Override the event observer's class
* Comment the observer from the 3rd-party's module

[Events and observers: Disabling an observer](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/events-and-observers.html#disabling-an-observer)

## 25

Which element below has effect on Tax and price of product when customer checkout?

* Customer billing address
* **Customer Group**
* base_price and price properties with the discounted price
* Customer shipping address

[What is a Customer Group in Magento 2 and What Do They Do?](https://www.customerparadigm.com/what-is-a-customer-group-in-magento-2/)

## 26

You are troubleshooting a product with a price of $25.
The special price is $20 and is active through 2017.
The tiered pricing has a discount for 2 or more products at $22.00.
Additionally, a catalog price url applies a 10% off discount.

What is the final price that is shown on the product page?

* $18.00
* $20.00
* **$22.50**
* $22.00

Из всех условий действует только каталожная скидка 10%, итого: 25 * 0.9 = 22.5

## 27

You have created a custom theme and need to customize the view/frontend/templates/product/list.phtml file in the Magento_Catalog module.
Where do you place this file inside your theme?

* **Magento_Catalog/templates/product/list.phtml**
* templates/product/list.phtml
* templates/Magento_Catalog/product/list.phtml
* Magento_Catalog/templates/override/product/list.phtml

[Theme inheritance: Override templates](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/themes/theme-inherit.html#theme-inherit-templates)

## 28

You need to add a sensitive configuration setting to store password. Where your customizations will be taking into account security?

* one more point....
* **Add encryptor in config.xml**
* Add a frontend_model to render configuration input with attribute 'type="password"'

## 29

You want to validate data before an EAV attribute for CategoryEntity saves.

* **Create backend model**
* Use eav_entity_attribute_save_before event
* Use catalog_category_save_before event
* Create beforeSave plugin Category model

[How to Manage EAV Attributes Including Interface/Source/Backend Structure in Magento 2](https://belvg.com/blog/how-to-manage-eav-attributes-including-interface-source-backend-structure-in-magento-2.html)

## 30

You are building a new module that needs to utilize a custom URL path like: /maps/{MAP_ID}

What steps do you take to accomplish this?

* Create an interface that extends RouterInterface details additional methods added.
* Create a plugin for the RouterList class to insert your router into the selection criteria.
* **Create a class that extends RouterInterface**
* **In di.xml, add your router to the RouterList class' routerList parameter.**

[Routing: Custom routers](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/routing.html#custom-routers)

## 31

How can you restrict access to extension attributes?

* You can utilize magneto ACL. Add inside extension_attributes.xml
* You need control it in you custom code on attributes loading
* Aminpanel -> Attributes -> List then edit access for each attribute then you need.
* Open entity and select role for each attribute

[EAV and extension attributes](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/attributes.html)

## 32

You are configuring a new entry in etc/adminhtml/system.xml that is a select / dropdown type.
What must the class for the source model extend or implement?

* Magento\Backend\Model\AbstractSource
* Magento\Framework\Source\OptionInterface
* Magento\Eav\Model\Entity\Attribute\Source\AbstractSource
* **Magento\Framework\Data\OptionSourceInterface**

## 33

Which of the following is NOT a native shipping method?

* **Freight**
* USP
* DHL
* Free Shipping

Для всего кроме Freight, есть шипинг модели:

* [USP](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Usps/Model/Carrier.php)
* [DHL](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Dhl/Model/Carrier.php)
* [Free Shipping](https://github.com/magento/magento2/blob/2.4/app/code/Magento/OfflineShipping/Model/Carrier/Freeshipping.php)

## 34

Which Magento 2 products are composite? (choose 3)

* Virtual
* **Grouped**
* Simple
* **Bundle**
* **Configurable**

Composite определяется в свойстве типа ["_isComposite"](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Product/Type/AbstractType.php#L41) по умолчанию оно false, и преопределяется в этих типах:

* [Grouped](https://github.com/magento/magento2/blob/2.4/app/code/Magento/GroupedProduct/Model/Product/Type/Grouped.php#L49)
* [Bundle](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Bundle/Model/Product/Type.php#L38)
* [Configurable])(https://github.com/magento/magento2/blob/2.4/app/code/Magento/ConfigurableProduct/Model/Product/Type/Configurable.php#L96)

## 35

How to ship an item to a different location?

* **Native feature Stores > Settings > Configuration > Sales > Multishipping Settings**
* Native feature in EE version Stores > Settings > Configuration > Sales > Multishipping Settings
* You need install 3rd party module

[Multishipping Settings](https://docs.magento.com/user-guide/configuration/sales/multishipping-settings.html)

## 36

A customer wants to add an extra block to product detailed page after SKU

* You cannot create any new blocks in PDP use plugin
* Add block to catalog_product_view.xml
* Override in theme product/view/form.phtml
* **Add custom layout container to catalog_product_view.xml after sku block**

## 37

What are GDPR and CCPA?

* **Legislation that regulates data protection and privacy in the European Union and in the US California**
* Services that magneto allow as out of the box integration
* Different approach for serve content to the client

## 38

You created a plugin for some method. Your business logic require doesn't call the original callable(proceed) method.
What happened? (choose 2)

* **"Subject" method won't be called**
* **It will prevent the execution of all the plugins next in the chain and the original method call**
* "Subject" method won't be called but next plugins will work as expected
* It will prevent the execution of all the plugins next around plugins but after plugins will be called

## 39

In your module, you use constant Magento\Catalog\Model\Product\Type::TYPE_SIMPLE.
Which type of dependency you should set in composer.json

* **Require**
* Suggest

[Module dependencies: Hard dependencies](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_depend.html#hard-dependencies)

## 40

You installed new module, added plugin, but it doesn't work.
Class:

```php
final class zz{
    final protected function a (){};
    public function authirize (){};
    private function b (){};
}
```

* Your module is disabled.
* Plugins doesn't work with final methods.
* **Plugins doesn't work with final methods and classes.**
* Need to add sequence in module.xml and it will work.

[Plugins (Interceptors): Limitations](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/plugins.html#limitations)

## 41

You have form input in UI component form.
How to add validation?

* Add custom js code if component is yours or create mixin for Magento 2 component
* Add validation in template as class
* Add node validation in the node argument
* **Add node validation and add rule in the node**

[Custom form validation](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/validations/custom-form-validation.html)

## 42

What is true for configuration cron "1 1 1 * *" in Magento?

* Implement once an hour
* Implement once a year
* **Implement once a month**
* Implement once a minute

[Cron: crontab](https://ru.wikipedia.org/wiki/Cron#crontab)

## 43

What interface should implement Each Action class?

* **Magento\Framework\App\ActionInterface**
* Magento\Routing\Api\ActionInterface
* Magento\Controlle\Api\ActionInterface
* Magento\Framework\App\Controller\ActionInterface

[Routing: Action class](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/routing.html#action-class)

## 44

Where Magento 2 storing category name?

* catalog_category_names
* **catalog_category_entity_varchar**
* catalog_category_entity_text
* catalog_category_entity

## 45

What is Magento vault?

* Storage for credit cards
* Storage for customer passwords
* **Storage for payment Token**
* Storage for cookies
* Storage for for 3rd party authentication API

[Adding vault integration](https://devdocs.magento.com/guides/v2.4/payments-integrations/vault/vault-intro.html)

## 46

Custom data is saved into database from ERP API. You have a block that renders that info on all pages on front. On some pages it is correct, on some is not.
When some changes are done in API, it talks Magento about it. What is wrong?

* Custom model, that saves date from API, is bugged.
* Block should implements IdentityInterface, return a value as a hash of data from API and clean caches when API talks about changes.
* **Block should implements IdentityInterface, return static cache key and clean caches when API talks about changes.**

## 47

What do extension attributes allow? (choose 2)

* Save automatically on action creation
* **Auto generating getters and setters for extension attributes**
* **Use join directive for add attributes to a collection**
* Control access via ACL

## 48

What does the EU cookie law require with regards to cookie consent?

* **You must obtain consent with clear and comprehensive information about the store and access of the cookies.**
* It is the same as the US: no consent is required.
* You must obtain consent to set cookies
* All cookie usage is banned unless you have obtained a license from the EU Bureau of Internet Regulations

## 49

Keeping maintability in mind, how do you customize the format of addresses in emails?

* Create a plugin for the MagentoSalesOrderAddress::getFormatted method.
* Set the new template in etc/order_addresses.xml
* Use di.xml to add a new address formatter to MagentoSalesModelOrderAddressFormatterList
* **Stores > Configuration > Customers > Customer Configuration > Address Templates**

## 50

Which one of the following class types directly charges a credit card when capturing an invoice in Magento admin?

* **PaymentMethod**
* Payment
* Order
* Invoice

## 51

You need to add a residential / commercial destination selector to the shipping address on the checkout.

What are the steps needed to add this selector?

* **Add new column to the sales_order_address table.**
* **Create a checkout_index_index.xml file and add the uiComponent details.**
* Override MagentoBlockCheckoutShippingAddress and add a getter for the selector value.
* Add a Javascript mixin to render the selector.

[Add a new field in address form](https://devdocs.magento.com/guides/v2.4/howdoi/checkout/checkout_new_field.html)
[Magento 2: Add a Custom Field to Checkout Shipping Address](http://techjeffyu.com/blog/magento-2-add-a-custom-field-to-checkout-shipping)

В обоих статьях нет пункта про checkout_index_index.xml там это добавляется через php, через лаяут как [здесь](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Captcha/view/frontend/layout/checkout_index_index.xml#L26)

## 52

Differences between developer and production mode? (choose 3)

* **Production - Does not allow you to enable or disable cache types in Magento Admin.**
* Production - Enables automatic code compilation.
* **Production - Serves static view files from cache only.**
* Developer - Prevents automatic code file compilation. New or updated files are not written to the file system.
* **Developer - Shows errors on the frontend.**

[About Magento modes](https://devdocs.magento.com/guides/v2.4/config-guide/bootstrap/magento-modes.html)

## 53

How to get product collection which IDs are in array(16,17,18,20) ?

* $collection->addFieldToFilter(['entity_id'=>['in'=>[16,17,18,19]]])
* $collection->addFieldToFilter(array('entity_id'=>array('gt'=>16,'lt'=>'19')))
* **$collection->addFieldToFilter('entity_id', array(16,17,18,19));**
* $collection->addIdToFilter(array(16,17,18,19));

Во втором аргументе передаётся условие, там можно явно указть ключ с нужным оператором, или передать только значение, в конечном итоге условие будет обрабатываться в адаптере БД, по дефолту MySQL, т.е. [`Magento\Framework\DB\Adapter\Pdo\Mysql::prepareSqlCondition()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/DB/Adapter/Pdo/Mysql.php#L2970). Если передать массив без указания оператора, то условия будет интерпретировано как 
```sql
entity_id = 16 OR entity_id = 17 OR entity_id = 18 OR entity_id = 19
```

## 54

What is different between `entitytype_save_after` and `entitytype_save_commit_after`? (Choose 2)

* **If _save_after fails, data will not be persisted in DB.**
* _save_after doesn't exist.
* Magento 2 fires only on of this method at the same time.
* **If _save_commit_after fails, data will be persisted in DB anyway.**

* entitytype_save_after — генерируется примерно [здесь](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Model/ResourceModel/Db/AbstractDb.php#L424) в транзакции сохранения, после сохранения в БД, но до завершения транзакции.
* entitytype_save_commit_after — генерируется примерно [здесь](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Model/ResourceModel/AbstractResource.php#L99) после завершения транзакции сохранения объекта.

## 55

How do you make a new category attribute available to be edited in the admin panel?

* **Add the new attribute to the category_form.xml uiComponent.**
* In the data setup script, add the attribute to the category's default attribute set.
* Create a plugin for the `Form` class that renders the category form and add the extra field.
* This is done automatically.

[Add a category attribute](https://devdocs.magento.com/guides/v2.4/ui_comp_guide/howto/add_category_attribute.html)

## 56

Choose the correct relationship between tax and discount in magento

* Discount is calculated before tax
* **"System > Configuration > Tax > Calculation Settings" allows to change the order of calculating tax and discounts**
* Tax is calculated before discount
* Tax and discount are calculated independently in 2 separate total

[General Tax Settings: Calculation Settings](https://docs.magento.com/user-guide/tax/tax-settings-general.html#calculation-settings)

## 57

You need to make some modifications to an entity before it is saved to the database. Your tool of choice is a plugin.

What are requirements for a plugin?

* The plugin class must not be marked abstract.
* **The plugin class must be specified in di.xml.**
* The plugin class must not inherit the targeted class.
* **The targeted method or class must not be marked final.**
* **The plugin method must begin with the type of the plugin.**

[`Plugins`](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/plugins.html)

## 58

Which of following caches are in Magento 2.2 Community? (choose 7)

* **Merged layout files**
* **Database DDL**
* Varnish
* **Translates**
* **Module config**
* **EAV**
* **FPC**
* **Blocks HTML output**
* Sessions

[Manage the cache: Overview of cache types](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-cache.html#config-cli-subcommands-cache-clean-over)

## 59

You have been tasked with writing an addition to Magento's CLI that will synchronize the database with an external source. Which directory should you put your new CLI command into?

* **/Console**
* /Command
* /Api/Console
* /CLI

[Create your component file structure](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/module-file-structure.html)

## 60

Does Magento 2 use cookies out of the box?

* No
* **Yes**

## 61

How to make attributes be able to be used in catalog price rules?

* All of them are incorrect
* Attributes added to products are not included in catalog condition
* Rewrite model catalog to add attributes
* **Edit attributes to allow display in catalog condition**

Поле `is_used_for_promo_rules` в `catalog_eav_attribute`. Можно поменять через админку в _Admin Panel — Stores — Attributes — Product_: Use for Promo Rule Conditions

## 62

You need to programmatically create a new customer attribute.

What steps are required to do this?

* Add the attribute to the customer_eav_attribute table.
* **Save the attribute.**
* **Create the attribute with Magento\Eav\Setup\EavSetup::addAttribute**
* **Specify the used_in_forms data for the attribute.**
* Set the source_model value for the attribute.

```php
    /**
     * @return AddPhoneAttribute|void
     * @throws \Magento\Framework\Exception\NoSuchEntityException
     * @throws \Magento\Framework\Exception\StateException
     */
    public function apply()
    {
        /** @var CustomerSetup $eavSetup */
        $eavSetup = $this->customerSetupFactory->create(['setup' => $this->moduleDataSetup]);

        $eavSetup->addAttribute(\Magento\Customer\Model\Customer::ENTITY, 'phone', [
            'label' => 'Phone',
            'input' => 'text',
            'type' => 'varchar',
            'validate_rules' => '{"max_text_length":255,"min_text_length":1}',
            'sort_order' => 120,
            'position' => 120,
            'visible' => true,
            'user_defined' => true,
            'unique' => false,
            'required' => false,
            'system' => false,
        ]);

        $phoneAttribute = $eavSetup->getEavConfig()->getAttribute(
            \Magento\Customer\Model\Customer::ENTITY, 'phone'
        );
        $phoneAttribute->setData('used_in_forms', ['adminhtml_customer']);
        $this->attributeResource->save($phoneAttribute);
    }
```

## 63

What is the difference between setting cacheable="false" on a block in layout XML attribute and the block's getCacheLifetime() === null?

* **cacheable="false" prevents the entire page from caching. getCacheLifetime() === null prevents the block from being cached, but it would still be cached by the full page caching mechanism**
* cacheable="false" affects the parent block or container. getCacheLifetime() === null prevents the current block from being cached.
* cacheable="false" and the effect of getCacheLifetime === null are the same.
* cacheable="false" prevents a block from being cached. getCacheLifetime() has been deprecated.

[Magento 2 Block Cache](https://www.mageplaza.com/devdocs/magento-2-block-cache.html#type-1)

## 64

You need to add a sensitive setting. How to do it? choose 2

* **Set setting as a obscure type**
* **Create Encryptor backend model**
* Create Encryptor frontend model
* Add plugin onSave attribute to encode and decode its value

[How to create a password type field in admin configuration in magento2](https://webkul.com/blog/how-to-create-an-obscure-field-in-admin-configuration-in-magento2/)

## 65

You want to sell a set of furniture. It is possible to order a table with adjustable size and color, change amounts and color of chairs.
Which is the product type you choose?

* Simple
* **Bundle**
* Configurable
* Grouped

[Bundle Product](https://docs.magento.com/user-guide/catalog/product-create-bundle.html)

## 66

At what URL would you visit the controller listed above
You see code in _etc/frontend/routes.xml_
```xml
<route id="mymodule" frontendName="user-subscriptions">
    <module name="MyCompany_MyModule" />
</route>
```
You have placed a controller in
Controller/Index/Subscribe.php
At what URL would you visit the controller listed above?

* /user-subscriptions/index/subscribe
* /mymodule/subscribe
* **/mymodule/index/subscribe**
* /user-subscriptions/subscribe

[Routing: Standard router](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/routing.html#standard-router)

## 67

Where are admin uiComponents located?

* view/ui_component/adminhtml/[COMPONENT NAME].xml
* view/components/adminhtml/[COMPONENT NAME].xml
* **view/adminhtml/ui_component/[COMPONENT NAME].xml**
* view/adminhtml/layout/[COMPONENT NAME].xml

## 68

You are creating an implementation of the grid UI component for a listing of product promotions in the backend. Where do you put it?

* **/view/adminhtml/ui_component**
* /view/admin/ui_component
* /view/adminhtml/grids
* /view/ui_component/adminhtml

## 69

How to get viewModel in a template?

* **$block->getViewModel()**
* $block->getData('view_mode;')
* $this->getData('view_model')
* $this->getViewModel()

[View models](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/view-models.html)

## 70

You want to convert data to lower case before an EAV attribute for ProductEntity saves.
What will you choose?

* **Create backend model**
* Use catalog_product_save_before event
* Use eav_entity_attribute_save_before event
* Create beforeSave plugin Product model

## 71

You are inspecting an existing Magento installation. Where do modules that affect the functionality of the Magento application reside? (choose 2)

* code
* module
* **app/code**
* **vendor**

[Module overview: Module location conventions](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_intro.html#m2arch-module-conventions-location)

## 72

Customer use a multi currency on website, but need to have a default price in the order export. What you need to get it in the generated file?

* Create own functionality to convert product price into default one.
* **Call method getBaseRowTotal().**
* Call method getRowTotal().

## 73

Where are the locations of a template in Magento 2? (Choose 2)

* /templates//view/frontend/templates/
* **`<theme_dir>`/`<namespace>`_`<module>`/templates/`<path_to_templates>`**
* `<theme_dir>`/templates/`<namespace>`_`<module>`
* **/view/frontend/templates/**

[Magento theme structure](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/themes/theme-structure.html)
[Create your component file structure](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/module-file-structure.html)

## 74

How to provide data from backend to UI component?

* **Implement DataProviderInterface and declare inside node in ui_component config**
* It is impossible without customization
* Create collection and add it inside node in ui_component config
* Create method getUiData in block then add it on frontend to window.myComponent = getUiData() ?>

[The DataSource Component](https://devdocs.magento.com/guides/v2.4/ui_comp_guide/concepts/ui_comp_data_source.html)

## 75

You added to default.xml
```xml
<referenceBlock name="content">
 <block name="myHeavyBlock" cacheable="false" template="havyBlocks/template.phtml"/>
</referenceBlock>
```
All pages become very slow. What happened?

* You cannot modify default.xml
* Block contain some "heavy" code
* **All pages become uncacheable**
* Error in layout

cacheable — отлючает кеширования для всех родительских блоков и отключает полнострничное кеширование на странице, а блоки присутвубщие в `default.xml` обычно присутвуют на всех страницах сайта, тем самым добавление не кеширумого блока таким образом отключило кеш для всех страниц.

## 76

A customer asked you to add the VAT number field to the shipping address form on checkout.
What steps should you perform to achieve this? (choose 3)

* **Add your field to JS layout via plugin** MagentoCheckoutBlockCheckoutLayoutProcessor::process
* Add you custom field to composite Magento\Checkout\Block\Checkout\Shipping\Form\Fields via di.xml
* **Add a JS mixin for Magento_Checkout/js/action/set-shipping-information to modify data submission**
* **Create extension attribute and add logic for process you field**
* Add your field to JS layout via plugin Magento\Checkout\Block\Checkout\Shipping\Form::render

[Add a new field in address form](https://devdocs.magento.com/guides/v2.4/howdoi/checkout/checkout_new_field.html)
[Magento 2: Add a Custom Field to Checkout Shipping Address](http://techjeffyu.com/blog/magento-2-add-a-custom-field-to-checkout-shipping)

## 77

How to insert static block in template file(.phtml)?

* **echo $this->getLayout()->createBlock('cms/block')->setBlockId('static_block_id')->toHTML();**
* echo $this->getLayout()->getBlockId('static_block_id')->toHTML();
* echo $this->getLayout()->createBlock('cms/block')->getBlockId('static_block_id')->toHTML();
* echo $this->getLayout()->setBlockId('static_block_id')->toHTML();

## 78

What is omnichannel retailing?

* Present on all or most social media channels.
* A multi-national / multi-lingual ecommerce setup.
* **Coordinated and operating across multiple means to access products or services.**
* Multiple sites / brands for the same offering.

## 79

What kind of subject is used to provide data, called from layout file?

* **Block**
* Model
* Resource Model
* Helper
* Controller

## 80

How do you enable Google Analytics eCommerce tracking in Magento?

* Store > Configuration > Sales > Google API > Google Analytics: Enable = Yes, Specify Account Number
* It is automatically installed and configured
* Just install a Google Analytics module
* System > Configuration > Google Analytics > Enabled = Yes

[Google Analytics](https://docs.magento.com/user-guide/marketing/google-universal-analytics.html)

## 81

What Magento layout XML instructions instruct Magento to render HTML output?

* referenceBlock
* **block**
* action
* container

[Layout instructions](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/xml-instructions.html)

## 82

You are customizing a third-party module and need to prevent an event handler from triggering.

Keeping upgradeability in mind, how do you do this?

* Create a plugin that will disable the 3rd-party's event observer method.
* Comment the observer from the 3rd-party's module.
* Override the event observer's class.
* **Create an observer in observer.xml with the same name and specify the disabled="true" attribute.**

## 83

What two places can change inventory levels?

* **QTY field on product edit page and update attributes (from product grid) > advanced inventory tab.**
* QTY field on product edit page and Inventory menu > Update.
* System > Inventory > Update Product Level and System > Inventory > Receive Shipment.

## 84

What files are required for a new custom theme? (choose 2)

* composer.json
* **theme.xml**
* etc/view.xml
* **registration.php**

[Create a new storefront theme](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/themes/theme-create.html)

composer.json — обязателен только если тема будет ставится через composer.

## 85

How to reduce a performance impact for classes with a big amount of dependencies in a 3rd party modules?

* Refactor code to reduce amount of dependencies
* **Add config for use proxy pattern via di.xml**
* Create plugin for __construct
* Create preference for slow class and implement lazy loading

[Proxies](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/proxies.html)
[Magento 2 Object Manager: Proxy Objects](https://alanstorm.com/magento_2_object_manager_proxy_objects/)

## 86

What minimum requirements for creating a custom block?

* Extends Magento\Framework\View\Element\AbstractBlock
* Extends Magento\Backend\Block\Template
* Implements toHtml() method
* **Implements Magento\Framework\View\Element\BlockInterface**

## 87

You use the method from the 3rd party module. But you check if module enabled.
Which type of dependency you should set in composer.json?

* Require
* **Suggest**

[Module dependencies: Soft dependencies](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_depend.html#soft-dependencies)

## 88

What change does Magento 2 introduce in regards to creating an account during checkout?

* Magento 2 does not allow guest checkout by default, but it can be turned on
* **Magento 2 allows a user to create an account after checking out**
* Magento 2 no longer allows guest checkout (due to higher incidents of fraud)
* Magento 2 allows guest checkout, but only when using the one-step-checkout template.

## 89

As you are assessing a 3rd-party module, you see this file: /Setup/InstallSchema.php. What is the intended purpose of this file?

* Sets up external API schema information.
* Installs schema-related data into the database.
* Initializes Magento configuration for the module.
* **Installs schema-related entities (tables, columns) into the database.**

[How to Create Setup, Upgrade, Uninstall Script in Magento 2](https://www.mageplaza.com/magento-2-module-development/magento-2-how-to-create-sql-setup-script.html)

Начиная с 2.3 используется [Declarative Schema](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/)

## 90

How to eliminate performance degradation when using constructor dependency injection?

* Use ObjectManager
* **Use Proxy**

[Proxies](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/proxies.html)
[Magento 2 Object Manager: Proxy Objects](https://alanstorm.com/magento_2_object_manager_proxy_objects/)

## 91

Which of the following features does Magento Community NOT have (when compared with Magento Commerce)?

* Full Page Caching
* Custom Design Configuration
* Search Term Customization
* **Visual Merchandiser**

[Visual Merchandiser](https://docs.magento.com/user-guide/marketing/visual-merchandiser.html)

## 92

The final price for the product on the product view page is....

* taken from the catalog_product_index_price_final_idx table
* pre-calculated in the products price attribute
* **calculated on-the-fly on php-level**
* taken from the catalog_product_index_price table

## 93

your webshop sells cars parts. customer wants to display all manufacturer list on cart page per each product.

* Add frontend model
* Add backend model
* Add UI component
* **Add source model**

## 94

Can Magento 2 work without cookies?

* **Yes, Magento 2 support cookie restricted mode.**
* No, without need customization.
* No, It is impossible.
* Yes, but will use localstorage instead.

[Cookie Restriction Mode](https://docs.magento.com/user-guide/stores/compliance-cookie-restriction-mode.html)

## 95

What only happens in the Production deploy mode?

* Static assets are created when the user downloads them.
* **Errors are only logged, not shown to the user.**
* Automatic code file compilation / generation happens as needed.
* Performance is faster because symlinks are established in the pub/static folder.

[About Magento modes](https://devdocs.magento.com/guides/v2.4/config-guide/bootstrap/magento-modes.html)

## 96

Which entity collects the grand_total and base_grand_total that the different total models collect?

* Checkout/cart
* Sales/order
* Sales/quote_address
* **Sales/quote**

## 97

What do extension attributes allow? (choose 2)

* Load automatically on action creation
* **Auto data hinting**
* **Clear declaration via XML**
* Improve performance on loading

## 98

You performing a code review on an existing module.
You see some interesting files in the Setup/ folder.
What files in the Setup/ does Magento understand? (choose 3)

* **Setup/Recurring.php**
* **Setup/InstallSchema.php**
* Setup/RecurringSchema.php
* Setup/InstallDetails.php
* **Setup/UpgradeSchema.php**

[How to Create Setup, Upgrade, Uninstall Script in Magento 2](https://www.mageplaza.com/magento-2-module-development/magento-2-how-to-create-sql-setup-script.html)

## 99

As you create a controller in Magento's adminhtml area, you must configure it to respond appropriately to the ACL.

Keeping simplicity in mind, what steps do you take to implement this?

* Create a plugin to ensure that the ACL resource is accessible.
* Check the authorization class' isAllowed method as the first check in the execute function.
* **Set the value of the ADMIN_RESOURCE constant to be the ACL resource.**
* Override _isAllowed and check the authorization class' isAllowed method for the ACL resource.

ACL проверяется в методе [`_isAllowed()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/App/AbstractAction.php#L211):
```php
protected function _isAllowed()
{
    return $this->_authorization->isAllowed(static::ADMIN_RESOURCE);
}
```

## 100

What is the minimum PHP version required for magento 2.2 installation?

* 7.1.0
* 5.6
* **7.0.13**
* 7.0.0

Для 2.3 — 7.2
Для 2.4 — 7.4.0

[Magento 2.4 technology stack requirements](https://devdocs.magento.com/guides/v2.4/install-gde/system-requirements-tech.html)

## 101

What files are required for a new theme?

* etc/view.xml
* composer.json
* **theme.xml**
* **registration.php**

[Create a new storefront theme](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/themes/theme-create.html)

composer.json — обязателен только если тема будет ставится через composer.

## 102

The store owner has custom system of rewards for the customer. They want to add Balance with reward points to the Customer Account Dashboard.
How do you customize it?

* Create UI component for show review rewarads
* Use observer render_account_dashboard_after
* Create plugin for CustomerAccount block and append you business logic directly
* **Add block and template to customer_account_index.xml**

## 103

you have webshope under developement. you are going to upgrade magento to new version. specify steps to do it:

* **composer require <magento_package> --no-update composer update bin/magento setup:upgrade**
* bin/magento maintenance:enable composer require <magento_package> composer update bin/magento maintenance:disable
* bin/magento maintenance:enablebin/magento cache:flushcomposer require bin/magento setup:upgradebin/magento maintenance:disable
* bin/magento maintenance:enablecomposer require <magento_package> bin/magento setup:upgradebin/magento maintenance:disable

[Upgrade Magento](https://devdocs.magento.com/guides/v2.4/comp-mgr/cli/cli-upgrade.html)

## 104

You have a text attribute. Your managers have access to edit products. Sometimes the value of attribute contain extra spaces when is rendered on front. How would you fix it?

* **add UI component to validate attribute value on beforeSave**
* add plugin on controller's save action, named beforeSave
* create backend model for attribute and remove spaces in beforeSave method

## 105

Are you required to include VAT in consumer purchases?

* Yes
* **No**

## 106

How to add product attribute that would store a static block reference ?

* **Add a source_model**
* Add a backend_model
* Add a frontend_model
* Attribute type should be text

## 107

Where are admin uiComponents located?

* view/components/adminhtml/[COMPONENT NAME].xml
* **view/adminhtml/ui_component/[COMPONENT NAME].xml**
* view/components/[COMPONENT NAME].xml
* view/adminhtml/layout/[COMPONENT NAME].xml

## 108

What ways are possible to wrap a block with an HTML tag?

* **Add a custom Layout XML container.**
* **Add the tag to the template.**
* Add a plugin for the afterHtml method and modify output.
* Listen to the core_block_html_output_after event and modify output.

## 109

Where can the PayPal express checkout button be displayed (choose 2)

* Checkout payment methods
* **Basket**
* **Product page**
* Category

## 110

How do you create a new cache type?

* **Create etc/cache.xml and specify a**
* Inject a new caching class into MagentoFrameworkAppCacheList
* Use a plugin to append to the method arguments for the MagentoFrameworkAppCacheList::collect method
* Add an event observer for the cache_type_collector observer.

[Create a cache type](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/cache/partial-caching/create-cache-type.html)

## 111

Can a customer include an image with their product review?

* Yes
* **No**

## 112

What is the difference between grand_total and base_grand_total?

* base_grand_total is calculated before tax, while grand_total is calculated after tax.
* base_grand_total is calculated in USD, while grand_total is calculated in other foreign currencies.
* base_grand_total is calculated when submitting orders, while grand_total is calculated when closing orders
* **base_grand_total is calculated based on the base currency, while grand_total is calculated based on the currency of the website's location where orders are submitted**

## 113

How to add and display a category attribute? (choose 2)

* It will be displayed automatically.
* Add use_in_forms data before save $attribute->setData('used_in_forms', ['category_edit']).
* **Creating a category_form.xml file in the view/adminhtml/ui_component directory in your module.**
* **Add attribute using MagentoEavSetupEavSetup::addAttribute via data patch.**

[Add a category attribute](https://devdocs.magento.com/guides/v2.4/ui_comp_guide/howto/add_category_attribute.html)

## 114

What Magento classes implement Magento\Framework\Controller\ResultInterface (choose 3)

* Magento\Framework\View\Result\Forward
* **Magento\Framework\Controller\Result\Redirect**
* **Magento\Framework\Controller\Result\Json**
* **Magento\Framework\View\Result\Page**
* Magento\Framework\Controller\Response\Redirect

## 115

How to get path to module directory?

* Use Magento\Framework\Filesystem\DirectoryList\Reader
* Use Magento\Framework\Module\Reader\Directory
* **Use Magento\Framework\Module\Dir\Reader**
* Use Magento\Framework\Filesystem\DirectoryList

[`Magento\Framework\Module\Dir\Reader`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Module/Dir/Reader.php)

## 116

What are the correct ways to call and initialize JavaScript in *.phtm templates?(choose 2)

* **Any tags with data-mage-init attribute e. g.**
* <script src="myscripts.js"></script>
* **<script type="text/x-magento-init"> ... </script>**
* <script>...</script>

[Four Ways to Add JavaScript to Magento 2 ](https://belvg.com/blog/4-way-to-add-javascript-to-magento-2.html)

## 117

What is the purpose of the getFlatColumns method in an EAV's Source model?

* **It returns columns that will be added to the EAV model's flat table.**
* It flattens an array attribute value into a JSON string.
* It returns a list of columns in the entity's _text table.
* It converts option values into row values for the attribute value tables.

## 118

Magento's JavaScript application has a means to instantiate a object through RequireJS. What is the correct node to do this?

* **<script type="text/x-magento-init"></script>**
* <script type="application/magento" src="path/to/script.js"></script>
* <script type="text/javascript"></script>
* <script type="text/require-js" module="path/to/module"></script>

[Four Ways to Add JavaScript to Magento 2 ](https://belvg.com/blog/4-way-to-add-javascript-to-magento-2.html)

## 119

Which products attributes are static? (choose 2)

* **attribute_set_id**
* **media_gallery**
* spectial_price
* category_ids

static атрибуты продукта:
* category_ids
* created_at
* has_options
* media_gallery
* required_options
* sku
* updated_at
static атрибуты категории:
* children_count
* level
* path
* position
static атрибуты кастомера:
* confirmation
* created_at
* created_in
* default_billing
* default_shipping
* disable_auto_group_change
* dob
* email
* failures_num
* firstname
* first_failure
* gender
* group_id
* lastname
* lock_expires
* middlename
* password_hash
* prefix
* rp_token
* rp_token_created_at
* store_id
* suffix
* taxvat
* updated_at
* website_id
static атрибуты адреса кастомера:
* city
* company
* country_id
* fax
* firstname
* lastname
* middlename
* postcode
* prefix
* region
* region_id
* street
* suffix
* telephone
* vat_id
* vat_is_valid
* vat_request_date
* vat_request_id
* vat_request_success

## 120

What is the order to generate 3 events: (1)sales_order_place_after, (2)sales_convert_quote_to_order,(3)sales_order_save_before

* 1->2->3
* **2->3->1**
* 1->3>2
* 3->2->1

# 121

You are tasked with building a new module to show product promotions to a customer. What two steps do you take to make the module active?

* php bin/magento deploy:mode:set developer
* php bin/magento sampledata:deploy
* **php bin/magento module:enable MyCompany_MyModule**
* **php bin/magento setup:upgrade**

[Enable or disable your component](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/enable-module.html)

# 122

There are some products, that always were visible. After product import, they disappeared on frontend. What is possible reason they are not shown in categories? (choose 3)

* Product out of stock
* **Invisibility in catalog, search**
* **Status disable**
* **Website Assignee**

# 123

A gift card is a product type?

* No
* Yes

[Gift Cards (EE)](https://docs.magento.com/user-guide/catalog/product-gift-card.html)

# 124

How to convert attribute value on save (e.g. from 1,2,3 to 1_2_3) taking into account the best practice, in case of invalid input exception needs be thrown?

* configure ui component
* plugin
* Override attribute class
* **create backend model**

# 125

To change the checkout mode from GUEST to REGISTER and vice versa, which event is needed?

* **checkout_allow_guest**
* is_allowed_register_checkout
* change_checkout_method
* is_allowed_guest_checkout

# 126

You need to add a fee (additional to tax and shipping) to every order placed.
In what file do you specify the new total?

* **etc/sales.xml**
* etc/di.xml
* etc/totals.xml
* etc/extension_attributes.xml

В файле _`<module-dir>`/etc/sales.xml_ регистируются новые тоталы:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Sales:etc/sales.xsd">
    <section name="quote">
        <group name="totals">
            <item name="custom_total" instance="Vendor\Module\Model\Totals\Custom" sort_order="500"/>
        </group>
    </section>
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
</config>
```

# 127

You want to add your custom data to Magento\Checkout\Model\Composite\Config\Provider:getSerializedCheckoutConfig().
What is the best way to customize it?

* Create after plugin
* Create around plugin
* **Add new item for configProviders in di.xml**
* Create preferance

в `di.xml`:
```xml
<type name="Magento\Checkout\Model\CompositeConfigProvider">
    <arguments>
        <argument name="configProviders" xsi:type="array">
            <item name="cc_card_config_provider" xsi:type="object">Magento\Payment\Model\CcConfigProvider</item>
        </argument>
    </arguments>
</type>
```

# 128

What additional tools are available in the backend and not in frontend? (choose 3)

* **URL secret keys**
* **ACL**
* Redis cache storage
* uiComponents
* **Global records search**

# 129

Where do you specify whether or not a customer can checkout as a guest?

* Install a module to enable this functionality
* In configuration, Checkout > User Types > Allow Guests to Checkout
* On the dashboard, check the Allow Guests to Checkout box
* **In configuration, Sales > Checkout > Allow Guest (Checkout)**

[Guest Checkout](https://docs.magento.com/user-guide/sales/checkout-guest.html)

# 130

Which table to store the relationship data between configurable product and it's simple product?

* catalog_product_link
* **catalog_product_super_link**
* both table catalog_product_superlink and catalog_product_relation
* catalog_product_relation

# 131

Your client current sells in North America with an English web site but would like to branch out internationally with a Spanish and French website. To maintain one admin panel for all languages and the same theme, you need to set up

* **One Magento installation, one website, three store views (one per language)**
* One Magento installation per language using automated import/export profiles to sync the product catalog
* One Magento installation, one website, one store view, two secondary languages
* Magento doesn’t handle languages by default, best approach is to use an extension

# 132

You want to add a discount when customers order more than 3 products.
How to do it?

* Create cart rule
* **Use Tired price via admin**
* Create plugin for AddToCartMethod and change price
* Use observer add_to_cart_before and change price

[Manage prices for multiple products: Manage tier prices](https://devdocs.magento.com/guides/v2.4/rest/modules/catalog-pricing.html#manage-tier-prices)

# 133

Customer has own ERP. Customer wants to get data about invoice in his own ERP. What approach you will choose?

* Add preference on Invoice model.
* Plugin the Invoice model.
* **Create observer for event register_invoice_after.**

# 134

When customers translate one word in three following position: table translation, i18n, which position will be displayed?

* All answers are correct.
* app/code/module/i18n
* app/etc/i18n
* **table translation**

[`Magento\Framework\Translate::loadData()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Translate.php#L218)
[Translations loading order](http://www.ajourquin.com/magento2/translations-loading-order/)

# 135

When we creates an attribute (price format) to product, where is it stored?

* catalog_product_entity_text
* **catalog_product_entity_decimal**
* catalog_eav_attribute
* catalog_product_entity

# 136

Discount "To fixed Ammount" 10$ means:

* None of the above is correct
* The minimum price is $10
* **Fixed Discount $10**
* The product is sold at $10

[Create a Cart Price Rule: Actions](https://docs.magento.com/user-guide/marketing/price-rules-cart-create.html#actions)

# 137

What interface should a frontend controllers action implement?

* Magento\Frontend\Controller\ControllerInterface
* Magento\Frontend\Controller\ControllerInterface
* Magento\Framework\App\ControllerInterface
* **Magento\Framework\App\ActionInterface**

[Routing: Action class](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/routing.html#action-class)

# 138

What ways are possible to wrap a block with an HTML tag?

* Listen to the core_block_html_output_after event.
* **Add the tag to the template.**
* Add a plugin for the afterHtml method.
* **Layout XML container**

# 139

Does Magento 2 support SOAP?

* No
* Yes

[SOAP Reference](https://devdocs.magento.com/guides/v2.4/soap/bk-soap.html)

# 140

When setData (' some', 'value') is called on an EAV entity and the entity is saved to the database?

* The 'value' of the attribute named T some' is saved in the eav attribute values table
* The 'value' of the attribute named 'some' is saved in the eav_values table
* **The 'value' of the attribute named ' some' is saved in one of the entity's tables depending on its datatype (for example, entityname_varchar)**
* The data will be stored in the EAV registry making ' some' 'value' available to the entity

# 141

You have an instance of the ProductCollection. What method do you call to load a subset of attributes for each product?

* $productCollection->addAttributeToRetrieve('attribute_name')
* $productCollection->loadAttribute('attribute_name')
* $productCollection->loadWithAttributes(['attribute_name'])
* **$productCollection->addAttributeToSelect('attribute_name')**

# 142

You have built a customization that changes a template in another 3rd-party module. That change is not taking effect. You check to ensure the template file path is correct—it is. Which two things are the problem? (choose 2)

* The block type is incorrect.
* **The layout XML file has the wrong name.**
* The template has invalid markup
* **The module's sequence is improperly configured**
* The template's file path is incorrect.

# 143

What key feature was added to Magento 2 CE?

* Downloadable products
* Search-term configuration
* Shipping labels
* **Command-line interface (ex. to help install modules)**

# 144

You are evaluating a 3rd-party module and you see some custom functionality that is executed in an observer of the checkout_cart_add_product_complete.It is obvious that this is supposed to run every time a product is added to the cart. Is there a problem?

* No problem: checkout_cart_add_product_complete is triggered as expected
* **Yes, problem. This is only triggered when the product is added to the cart on the frontend and in no other case**
* Yes, problem. This event doesn't exist.
* Yes, problem. This event is used only triggered in the backend.

# 145

What is the first argument in plugins?

* Result of original method execution
* **$subject (e. g. Instance of plugged class)**
* $config (e. g. Instance of plugin config)

# 146

You are tasked with installed a new 3-party module to show product promotions to a customer.
What two steps do you take to make the module active? (choose 2)

* php bin/magento sampledata:deploy
* **php bin/magento module:enable MyCompany_MyModule**
* php bin/magento deploy:mode:set developer
* **php bin/magento setup:upgrade**

[Enable or disable your component](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/enable-module.html)

# 147

Which tags below are not used in email template? (choose 2)

* {{depend ?}} ? {{/depend}}
* **{{model ?}}**
* {{store ?}}
* {{var ?}}
* **{{skin ?}}**
* {{if ?}} ? {{else}} ? {{endif}}

[Email templates](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/templates/template-email.html)

# 148

In Magento Enterprise, banners can be displayed based on (select all that apply):

* Tax Class
* Payment Method
* **Catalog Price Rules**
* **Shopping Cart Price Rules**
* **Customer Segment**

[Dynamic Blocks](https://docs.magento.com/user-guide/cms/dynamic-blocks.html)

# 149

How to upgrade Magento 2 CE for specific version locally(in developer mode) (Choose 2)

* **composer require magento/product-community-edition:**
* **bin/magento setup:upgrade**
* bin/magento setup:download:upgrade
* bin/magento server:upgrade

[Upgrade Magento](https://devdocs.magento.com/guides/v2.4/comp-mgr/cli/cli-upgrade.html)

# 150

Magento has several ways to notify a custom module when changes are made in the database. What are they? (choose 3)

* Setup an extension attribute
* **Create a watcher in etc/mview.xml**
* **Event listeners**
* **Plugins**
* Build a custom implementation around Zend_Adapter

# 151

You want to prevent the collection loading in the specific method.
You have some condition(for example Customer Groupe)
What is the best way to customize it
```php
public function loadingFoo($bar)
{
    return $this->collection->load();
}
```

* Preference
* After Plugin
* **Around Plugin**
* Before Plugin

# 152

Which product types implement a parent-child relationship between product entities?(choose 3)

* Downloadable
* Virtual
* **Configurable**
* **Bundle**
* **Grouped**
* Simple

# 153

How to create a Custom Order Status? (choose 2)

* Use di.xml for adding status item to MagentoSalesApiOrderStatusComposite.
* **Assign Status to State in admin Stores -> Order Status -> Assignment Information.**
* Create plugin for show new order status.
* **Create new status in admin Stores -> Order Status -> Create New Status.**

[How to Create Custom Order Status in Magento 2](https://www.mageplaza.com/kb/how-to-create-custom-order-status-magento-2.html)

# 154

A module contains the following files:

ModuleA/
|-- etc/
 |-- di.xml
 |-- frontend/
   |-- di.xml

When a visitor is browsing the frontend of the website, what happenes if a similar XML directive is specified in both di.xml files above?

* An error is thrown: Conflicting values are found in etc/di.xml and etc/frontend/di.xml
* **The two di.xml files are merged with etc/frontend/di.xml taking precedence.**
* etc/di.xml is used instead of etc/frontend/di.xml.
* etc/frontend/di.xml replaces etc/di.xml.

# 155

Assume you have saved a product in the Catalog. Will your last changes be taken into account for rule-based related products?

* Only if you go to the "Catalog/Rule-based Product Relations" page, and hit "Reindex All" button.
* **Yes, it will be done automatically by an observer on the catalog_product_save_after event.**
* Yes, rule-based related products are calculated on-the-fly.
* Only if you clear the cache.

# 156

Where do you set default config values for system configs?

* di.xml
* system.xml
* **config.xml**
* default.xml

[How to Set Default Values for Magento 2 System Configuration](https://meetanshi.com/blog/set-default-values-for-magento-2-system-configuration/)

# 157

What the requirement for adding an extension attribute to an entity?

* Entity has to have public function getExtensionAttributes();
* Entity has to call event add_assigned_extension_attribute
* Entity has to extend Magento\Framework\Model\ExtensibleData\AbstractProduct
* **Entity has to implements Magento\Framework\Api\ExtensibleDataInterface**

# 158

By default you have product url:
Example: http://mystore.com/helena-hooded-fleece.html

You have requirement including category path for all products
Example: http://mystore.com/clotheth/hooded/helena-hooded-fleece.html

* Change url attribute for product and add category to url-key
* Create plugin for getCategoryUrl
* Create custom router
* **Change Stores > Settings > Configuration > Categories Path for Product URLs**

[Catalog URLs: Configure catalog URLs](https://docs.magento.com/user-guide/catalog/catalog-urls.html#configure-catalog-urls)

# 159

What is the Visual Merchandiser?

* It is a tool that links banner ads with products
* It is a tool to structure and organize categories and CMS pages.
* **It is a tool to position products in the category listing.**
* It is a tool used to direct customers through a specific customer journey

# 160

You need to show some additional data when it calls API method on a customerRepository, but not save it in database. What you will do? choose 3

* add plugin on afterLoad
* **if getExtensionAttribute returns null, create new instance**
* **set extensionAttribute to CustomerInterface's instance**
* **add plugins after* on all methods that return CustomerInterface**

# 161

Which native Magento products are also composite products?

* **Grouped**
* Kit
* Simple
* **Bundle**

# 162

Which of the following operations is most impacted (in time required) by a large number of products and stores?

* saving product in the admin area
* loading a simple product page on the front
* **adding new attributes to be used in flat catalog**
* importing tax rates to the database

# 163

To delete button that is available on the system's form as button Back, Save, Save and Continue, Delete - we use the following way

* All are correct
* _dropButton ('name_button');
* _deleteButton ('' name_button ');
* **_removeButton ('name_button');**

# 164

You are configuring a new entry in etc/adminhtml/system.xml that is a select / dropdown type.
What must the class for the source model extend or implement?

* **Magento\Framework\Data\OptionSourceInterface**
* Magento\Eav\Model\Entity\Attribute\Source\AbstractSource
* Magento\Backend\Model\AbstractSource
* Magento\Framework\SourceOptionInterface

# 165

What framework does Magento 2 use for its REST API?

* **Swagger**
* Apigility
* API Platform
* Apigee

# 166

Describe 2 cases when need to create your own block class

* **If the template is determined dynamically at runtime (different item renderers)**
* When you want to render custom data, that is not present in default block, in template
* When you create a custom template and adds to page
* **You need to create block with non-default caching tags**

# 167

You added 3 product to the cart. What is the price will be final?

BasePrice - 50
SpecialPrice - 60
TierPrice for 2 products and more - 57
CatalogPrice Rule applied for 10% discount

* 60 x 3 x 0.9 = 162
* 57 x 3 = 171
* 57 x 3 x 0.9 = 153.9
* **50 x 3 x 0.9 = 135**

# 168

When declaring a field in system configuration in Magento ( in file system.xml), which input types need to declare source_model? ( Choose 2)

* text
* date
* **multiselect**
* **select**
* textarea

# 169

What is the address in which tax is calculated?

* Billing Address
* Shipping Address
* **According to config in admin Sales/Tax/Calculation Settings/Tax Calculation Based On**
* Shipping Origin

[General Tax Settings: Calculation Settings](https://docs.magento.com/user-guide/tax/tax-settings-general.html#calculation-settings)

# 170

Which tag below is used to check config in layout?

* depends
* action
* storeconfig
* **ifconfig**

# 171

What type of authentication does Magento 2's API use?

* REST
* **OAuth 1.0a**
* OAuth 2.0
* JSON 5.4

[OAuth-based authentication](https://devdocs.magento.com/guides/v2.4/get-started/authentication/gs-authentication-oauth.html)

# 172

You extended layout of 3rd party module.
Which type of dependency you should set in composer.json?

* **Suggest**
* Require

[Module dependencies: Soft dependencies](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_depend.html#soft-dependencies)

# 173

You are creating a new module to extend the functionality of the Magento application.
What files are required?

* **etc/module.xml, registration.php**
* Setup/Patch/InstallData.php, etc/config.xml, registration.php
* etc/module.xml, registration.php, etc/config.xml
* etc/config.xml, registration.php, composer.json

# 174

How to set order for a magento collection? (choose 2)

* **$coll->setOrder('entity_id','desc')**
* $coll->getSelect()->order('entity_id','desc')
* **$coll->getSelect()->order('entity_id desc')**
* $coll->setOrder('entity_id desc')

# 175

How to create a new indexer?(choose 3)

* **MagentoFrameworkIndexerActionInterface and MagentoFrameworkMviewActionInterface**
* **Add etc/ mview.xml in your module**
* Add your indexer as argument in MagentoFrameworkIndexerList via di.xml
* Add config.xml with default configuration of you indexer
* **Add etc/indexer.xml in your module**

[Adding a custom indexer](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/indexing-custom.html)

# 176

Magento 2 includes a responsive theme (Luma). Responsive web design is:

* An approach to web design which heavily utilizes Javascript to provide increased user interaction capabilities. Magento 2 has a WYSIWYG editor so that store owners can utilize plug and play responsive design and feedback elements.
* **An approach to web design where the same design can be viewed on different devices. Elements are dynamically resized and will rearrange or even be hidden based on the user’s device and resolution. A separate site for mobile users is no longer needed**
* A feature which can be activated for a specific theme in Magento which detects the user’s device (desktop, tablet, mobile phone) and routes the user to the proper website for their device.
* An approach to web design based on user feedback. A minimum survey sample rate of 1 per 100 users is recommended, with some businesses regularly getting feedback from as many as 1 in 10 site users.

# 177

What is purpose of the category with ID 0?

* it is used to show partial and full category tree.
* it is used to show category tree in admin.
* **used to be a parent for all other categories**
* __used to search categories in code. No other purpose__

**???**

# 178

What kind of Catalog Input Type for Store Owner must you set to make the attributes filterable in Layered Navigation? (Choose 3)

* **Price**
* **Multiple Select**
* Text
* **Dropdown**

# 179

A merchant asks you to create a module that is able to process URLs with a custom structure that can contain any combination of a product type code, a partial name, and a 4-digit year in any order.

The request path will look like this: /product/:type-code/:name-part/:year

Which layer in the Magento request processing flow is suited for this kind of customization?

Question from https://u.magento.com/free-study-guide

* **Router**
* HTTP Respons
* Front controller
* Action controller

# 180

What is the difference between varchar and static attributes?

* According to attribute they are saved to *_text table, _varchar tables
* **Static attribute is saved to _entity tables**
* Some entity use only static attributes, but some only varchar attributes
* No difference

# 181

What does FrontController do in Magento2?

* Redirect users according to UrlRewrites
* Maps you request to Controllers
* **Composite that searches through a list of router according to order**
* Converts Request to Magneto RequestInterface

[Routing: FrontController class](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/routing.html#frontcontroller-class)

# 182

You want to filter data from ProductRepository::getItems()
What will you do?

* **Utilize SearchCriteriaBuilder**
* Use ProductCollection instead of Repository
* Call ->filterData()
* Create plugin for getItem

[Searching with Repositories](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/searching-with-repositories.html)

# 183

You are building an tool that imports products from an ERP.
There are 20 columns of additional information that are associated with each product.
This extra information must also be associated with an update time to know when to refresh the data.
Keeping maintainability in mind, how do you build this into Magento?

* **Utilize an extension attribute.**
* Create 20 EAV attributes and check their updated_at column.
* Create a separate model and build code to associate the two record types.
* Override the Product model and add the fields.

# 184

How do you create a new cache type?

* Add an event observer for the cache_type_collector observer.
* **Create etc/cache.xml and specify a `<type name="cache_name" instance="ClassPath">.</type>`.**
* Inject a new caching class into MagentoFrameworkAppCacheList.
* Use a plugin to append to the method arguments for the MagentoFrameworkAppCacheList::collect method.

[Create a cache type](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/cache/partial-caching/create-cache-type.html)

# 185

What category path is the category that is the closest to the top of the tree, but still is visible on Layered Navigation?

* 1/2
* 1
* 0
* **1/2/8**

# 186

You need to programatically create a new customer attribute that would be editable in admin panel.
What steps are required to do this? (choose 2)

* **Create the attribute with MagentoCustomerSetupCustomerSetup::addAttribute**
* **Specify the used_in_forms data for the attribute**
* Set the source_model value for the attribute
* Create the attribute with MagentoEavSetupEavSetup::addAttribute
* Add the attribute to the customer_eav_attribute table

# 187

An order has the additional attribute "Extended Guarantee Period". You want to show it as a column inside order history in Account Dashboard.
How to add it?

* **Add a child block to container inside sales_order_history.xml layout.**
* Create plugin for OrderHistoryDataProvider.
* Add mixin for order-history.js.
* Rewrite sales_order_history.phtml template.

# 188

You added a configurable product to the cart.
You see configurable thumbnails image in cart.
How to change it?

* **Change config to Sales -> Checkout -> Shopping Cart -> Configurable Product Image -> Product Thumbnail Itself.**
* Create mixin for cart item render and change it on frontend.
* Change item render via layout.xml.
* Create plugin aroundGetThumbnail for cart item block.

# 189

If your client has two websites that sell different products yet wants to share a Magento installation for Admin purposes, you will need to set up:

* **One Magento installation, two websites and a store view for each website**
* Two Magento installations
* One Magento installation, one website, two merchant accounts and link them together
* One Magento installation, one website, one store view, and 2 display URL’s

# 190

You have already implemented custom module, but now you need to add new order attribute, to save the custom data

* __use InstallData.php and MagentoEavSetupEavSetup->addColumn('sales_order_item', ...)__
* use InstallData.php and MagentoEavSetupEavSetup->addAttribute(typeId, code, attrConfig)
* **use UpgradeData.php and MagentoSalesSetupSalesSetup->addAttribute(typeId, code, attrConfig)**
* use UpgradeData.php and $setup->getConnection()->addColumn('sales_order_item', ...);

???

# 191

You are adding an extension attribute to the CustomerInterface class.
You have specified data for the extension attribute, but when you check the database, nothing has been saved.
Why is that?

* **Extension attribute data is not automatically persisted to the database.**
* You need to ensure that the extension attribute getter and setter exists in CustomerExtensionInterface.
* You need to add a node to the extension attribute XML details.
* The appropriate columns in customer_entity have not been created.

# 192

Customer wants to sale only one quantity of each product per one customer. So the one customer can buy 2 different products, but each item couldn't have quantity more then 1.

* Add preference on MagentoCatalogInventoryModelStockStateProvider and rewrite quantity checking functions
* Add plugins on "Add to cart" and "Update shopping cart" actions
* Add an observer on "sales_quote_add_item" event
* **Set appropriate setting in Configuration/Catalog/Inventory/Product Stock Options/Maximum Qty Allowed in Shopping Cart**

[Inventory: Product Stock Options](https://docs.magento.com/user-guide/configuration/catalog/inventory.html#product-stock-options)

# 193

How can Customer Groups impact prices? Select all that apply. (choose 3)

* **Catalog Price Rules can be associated with a customer group.**
* Product special prices can be associated with a customer group.
* **Product tiered pricing can be associated with a customer group.**
* **Cart Price Rules can be associated with a customer group.**

# 194

What additional tools are available in the backend and not in frontend?

* **ACL**
* **URL secret keys**
* **Global records search**
* Redis cache storage
* uiComponents

# 195

Applying the shopping cart rule's action affects the quote item by setting the quote item's...

* base_price and price properties with the discounted price
* **base_discount_amount and discount_amount with the discount applied to original price**
* base_price_incl_tax and price_incl_tax properties with the discounted price
* base_row_total and row_total properties with the discounted price

# 196

What is the minimum requirement to create a CLI command?

* **Need to declare command by adding an argument to MagentoFrameworkConsoleCommandListInterface**
* Set the name of command in the command's constructor method
* Need to declare command in config.xml by xpath console/commands//
* Class should implement an interface MagentoFrameworkConsoleCommandInterface
* **Class should have an "execute" method**

[How to add CLI commands](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/cli-cmds/cli-howto.html)

# 197

Customer wants to show a welcome message with a user's name on all pages in the header.

* Override a header template in custom theme and render user's name if he is logged in
* Add JS Ajax to load user's name
* **Utilize a private content conception**
* Create own template and add it into header via layout xml injection

# 198

You are building a new module to add extra functionality to the Magento application.

What files are required?

* **etc/module.xml**
* composer.json
* Setup/InstallSchema.php
* etc/config.xml
* **registration.php**

# 199

Does Magento 2 have a “Saved Credit Card" payment method?"

* Yes
* No
* **Customer can save their credit cards tokens in 'Saved Payment Methods' which are available only for payment methods such as Braintree, that offer a secure vault.**

# 200

What only happens in the Production deploy mode?

* Static assets are created when the user downloads them
* Automatic code file compilation / generation happens as needed
* **Errors are only logged, not shown to the user**
* Performance is faster because symlinks are established in the pub/static folder

# 201

A customer wants to change the 404 pages completely
How to perform it?

* Add plugin aroundToHtml to change html output
* change 404.phtml
* **Add item for Magento\Framework\App\Router\NoRouteHandlerList in the di.xml**
* Add plugin afterToHtml to change html output

# 202

Which one of the following EAV attribute types may be used for layered navigation in native Magento?

* Varchar
* **Select**
* Int
* Text

# 203

UpgradeSchema.php only provides one method: upgrade.
What method do you use to determine what code to execute for what method?

* Put the updates into a method named containing the version number that the module is being upgraded to.
* **Use PHP's version_compare method to compare the $context->getVersion() with the required version.**
* Check the etc/module.xml XML file for the version number.
* Use the $setup->compareVersion method.

# 204

What is a billing agreement?

* An agreement to pay for recurring virtual products, such a monthly fee charged for access to a member’s only website
* **An agreement between your customer and the payment service provider that, once in place, allows your customer to checkout without entering payment information for each purchase**
* An alternate payment method
* A B2B agreement where one company provides billing services for another.

# 205

What is the difference in the effect of calling the invoice capture () method versus the invoice pay () method?

* pay () will trigger the payment and capture!) will not
* The difference is determined by the payment method implementation
* No difference: pay () will always call capture ()
* **capture () will trigger the payment and pay () will not**

# 206

Which statement correctly describes order state and order status?

* An order doesn't have a status, only a state. Status is a property of an invoice, shipment, and credit memo.
* **The status is a child of the state**
* State and status are independent properties of the order.
* State represents the general state of the order, while status works on item level.

# 207

What product types do not need to ship?(choose 2)

* Grouped
* **Virtual**
* Simple
* **Downloadable**
* Bundle
* Configurable

# 208

You are implementing customization of the sales management within a module MyCompany_MySalesProcess. You have created several event observers to add custom functionality. Each observer is a separate class, but they require some common functionality.

How do you implement the common functionality in the event observers, keeping maintainability and testability in mind?

* You create an abstract class AbstractObserver with the common methods and extend the observer classesfrom it.
* **You create a regular class implementing the common functionality as public methods and use constructor injection to make them available to the observers.**
* You create a regular class implementing the common functionality as public static methods and call those from the observers.
* You create a trait with the common methods and use the trait in the observer classes.

# 209

What pattern are you use each time when you adding something to di.xml

* data injection
* **dependency injection**
* dependency inversion
* developer inspection

# 210

While integrating a merchant’s product information management system with Magento, you create a module MyCompany_MerchantPim that adds a catalog product EAV attribute pim_entity_idprogrammatically.

In which type of setup script do you create the EAV attribute?

* Setup/UpgradeSchema.php
* Setup/InstallEntity.php
* **Setup/UpgradeData.php**
* Setup/InstallSchema.php 

# 211

You need to handle url like https://domain1.com/rest/v1/custom. How to do it?

* Create a custom router
* **Add configs in webapi.xml**
* Create custom controller and action.
* Utilize url_rewrite

# 212

As you create a controller in Magento's adminhtml area, you must configure it to respond appropriately to the ACL. Keeping simplicity in mind, what steps do you take to implement this?

* Check the authorization class' isAllowed method as the first check in the execute function
* Override _isAllowed and check the authorization class' isAllowed method for the ACL resource
* **Set the value of the ADMIN_RESOURCE constant of your class to be the ACL resource**
* Create a plugin to ensure that the ACL resource is accessible

# 213

You are inspecting an existing Magento installation.

Where do modules that affect the functionality of the Magento application reside?

* **app/code**
* **vendor**
* modules
* code

# 214

What steps you should perform for adding viewModel to the block.(choose 2)

* Add via di.xml
* **Create class that implements Magento\Framework\View\Element\Block\ArgumentInterface**
* **Add via Layout**
* Extends MagentoBackendBlockTemplate

# 215

What is a database sharding?

* Multiple websites or codebases accessing the same database
* Replicating a database across multiple locations
* **Splitting a database into multiple shards or pieces.**
* Creating a new database shard for every customer session.

# 216

Advantages of using EAV for a merchant

* Better data type validation
* Better performance because data separated in different tables
* **Changing attributes without altering DB schema**

# 217

You need to create a custom price calculator for simple products. You have already creates the new price model.

Keeping simplicity in mind, what additional steps do you take to implement this?

* Update the frontend templates to use the new price methods.
* **In your product_types.xml module, reference the simple product type, and set a value for the priceModel attribute.**
* **Make your new price model extend MagentoCatalogModelProductTypePrice.**
* Create a plugin for the getPriceModel() method and return your price model.

# 218

Magento's Product repository uses the getList($filter) method retrieve a list of products.
As you are preparing to retrieve a list of products, you need to specify what products to retrieve.
Which of the following classes will help you formulate the filter?

* Magento\Framework\Api\RepositorySearchBuilder
* **Magento\Framework\Api\SearchCriteriaBuilder**
* Magento\Framework\Api\App\SearchBuilder
* Magento\Framework\Api\SearchFilterBuilder

[Searching with Repositories](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/searching-with-repositories.html)

# 219

Magento area types are(choose 3):

* adminhtml, registered-users
* **webapi_soap, frontend**
* **adminhtml, crontab, graphql**
* **base, webapi_rest**
* base, guests

# 220

Where do you configure design options in Magento 2?

* **Content > Design > Configuration**
* Content > Configure Design Parameters
* Marketing > Site Look and Feel
* System > Configurations

[Design Configuration](https://docs.magento.com/user-guide/design/configuration.html)

# 221

What types of subject does Magento create cache for?(Choose 2)

* Controller
* Block's $data property
* Model
* **Block Output HTML**
* **Module Configuration**

# 222

You need to create a variation of the 2columns-left page layout. This new layout is named text.

How do you instruct Magento regarding the new layout type?

* Create a new page_layout.xml in the applied theme with .
* Insert text in to the layouts table.
* **Create view/frontend/layouts.xml with a node.**
* Create view/frontend/layout/text.xml, with .

# 223

When does an inventory level decrease through the ordering process?

* When the order is shipped.
* When the item is added to the shopping cart.
* When the order is invoiced.
* **When the order is submitted.**

Сток декрементится при ивенте ["sales_model_service_quote_submit_before"](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Quote/Model/QuoteManagement.php#L555) в [`Magento\CatalogInventory\Observer\SubtractQuoteInventoryObserver::execute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogInventory/Observer/SubtractQuoteInventoryObserver.php#L58) при создании заказа.
При отмене заказа он инкрементится при ивенте ["sales_order_item_cancel"](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/Model/Order/Item.php#L410) в [`Magento\CatalogInventory\Observer\CancelOrderItemObserver::execute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogInventory/Observer/CancelOrderItemObserver.php#L55). Происходит с помощью метода [`Magento\CatalogInventory\Api\StockManagementInterface::backItemQty`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogInventory/Model/StockManagement.php#L174)

# 224

Which products attributes are static? (choose 3)

* **name**
* **weight**
* url
* **price**

???

# 225

You creating customer address custom attributes. Which config you should add that allows you to edit the attribute in admin? (choose 2)

* **used_in_forms = 'admin'**
* **is_system=0**
* editable=1
* Magento adds it by default

# 226

What are valid plugin types? (choose 3)

* Instead
* **Around**
* **Before**
* Along
* **After**

# 227

Magento uses the concept of service contracts to create a blueprint for modules to communicate with each other.
Where are these service contracts stored in a module?

* Service
* Model
* **Api**
* Api/ServiceContracts/

# 228

Customer has one website and English, Italy, Spain stores. He wants to add a 20% discount for all products on Spain store:

* **Customization is required. Magento supports discounting only per website scope**
* It could be enabled via store configuration
* Catalog price rules have option to specify the store in "Apply To Stores" field

# 229

How do you enable customer comments on orders in Magento 2?

* Customer comments are automatically enabled.
* Customer comments must be turned on (via Configuration > Sales > Enable Order Comments) and are only available in One Step Checkout
* Customer comments must be turned on (via Configuration > Sales > Enable Order Comments)
* **Customer comments require a custom extension**

# 230

What can be done with etc/di.xml? (choose 3)

* **Replace constructor arguments**
* Register an event listener
* **Create virtual types**
* **Create a preference for Magento to replace a class lookup or specify a concrete class for an implementation**
* Replace configuration values in system configuration.

# 231

Your client has a single catalog but sells on two different domains.
While the design and product selection are different between two sites, the biggest difference between the two sites happens to be the shipping and payment methods. What would be better?

* **One Magento installation, two websites, and a store view for each.**
* One Magento installation, one website, one store view, along with a multi-payment and multi-shipping extension.
* Two Magento installations.
* One Magento installation, one website, two store views.

# 232

What is the factors that make Magento website slow?

* Huge XML trees built up for layout configuration, application configuration settings
* **Place many orders at the same time.**
* **EAV structure of magento database, even for retrieving single entity the query becomes very complex**
* Magento's template system involves a lot of recursive rendering

# 233

You need to create a custom price calculator for simple products.
You have already creates the new price model.
Keeping simplicity in mind, what additional steps do you take to implement this? (choose 2)

* Create a plugin for the getPriceModel() method and return your price model.
* Update the frontend templates to use the new price methods
* **In your product_types.xml, reference the simple product type, and set a value for the priceModel attribute.**
* **Make your new price model extend MagentoCatalogModelProductTypePrice**

# 234

What are the default required attributes for creating a product?

* Product Name, Product Description, Status
* SKU, Visibility, Status
* Inventory level, Product Name, URL Key
* **Price, Product Name, SKU**

# 235

How to add a custom router?(choose 2)

* **Implement RouterInterface**
* Crete plugin after for getRouters() in MagentoFrameworkAppRouterList
* Use event routing_process_get_routes_after
* **Add you route via di.xml to RouterList**

[How to Create Custom Router in Magento 2](https://belvg.com/blog/how-to-create-custom-router-in-magento-2.html)

# 236

You have created a new product type, sample, and need to customize how it renders on the shopping cart page.

Keeping maintainability in mind, how do you add a new renderer?

* **Create the layout file, checkout_cart_item_renderers.xml, reference the checkout.cart.item.renderers block and add a new block with an as="sample" attribute.**
* Create the layout file, checkout_cart_index.xml, and update the cart page's uiComponent to appropriately render the sample product type.
* Create the layout file, checkout_cart_index.xml, and reference the checkout.cart.renderers block and add a block with the as="sample" attribute.
* Override the cart/form.phtml template and add logic for the sample product type.

# 237

A customer wants to change the 404 pages content
What is the easiest way to do it?

* Use UI component
* Change web-server config to show custom html
* **Create new cms_page and assign it in the system config**
* Change 404.phtml

# 238

You want to prevent execution of some method. What is the best way?

* **Add plugin around**
* Add plugin before
* Add observer
* Add plugin after

# 239

You need access to some data in myTemplate.phtml.
What is the recommended approach?

* dataProvider
* dataController
* **viewModel**
* Block

[View models](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/view-models.html)

# 240

You created a plugin for ProductRepositroyInterface.
Which type of dependency you should set in composer.json?

* Suggest
* Require

[Module dependencies: Soft dependencies](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_depend.html#soft-dependencies)
