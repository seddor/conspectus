# Demonstrate ability to customize customer functionality

>Describe how to add or modify customer attributes.

>Describe how to extend the customer entity. How would you extend the customer entity using the extension attributes mechanism?

>Describe how to customize the customer address. How would you add another field into the customer address?

>Describe customer groups and their role in different business processes. What is the role of customer groups? What functionality do they affect?

>Describe Magento functionality related to VAT. How do you customize VAT functionality?

## Describe how to add or modify customer attributes.

[Customer Attributes (EE) (User guide)](https://docs.magento.com/user-guide/stores/attributes-customer.html)

В Enterprice Edition атрибуты можно редактировать из адинки.

В Community Edition атрибуты можно изменять/добавлять только программно с помощью [дата патчей](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/data-patches.html)

```php
<?php
namespace Oggetto\Exam\Setup\Patch\Data;

use Magento\Customer\Setup\CustomerSetup;
use Magento\Customer\Setup\CustomerSetupFactory;
use Magento\Framework\Setup\ModuleDataSetupInterface;
use Magento\Framework\Setup\Patch\DataPatchInterface;
use Magento\Eav\Api\AttributeRepositoryInterface;
use Magento\Customer\Model\ResourceModel\Attribute as AttributeResource;

class AddPhoneAttribute implements DataPatchInterface
{
    /**
     * @var ModuleDataSetupInterface
     */
    private $moduleDataSetup;
    /**
     * @var CustomerSetupFactory
     */
    private $customerSetupFactory;
    /**
     * @var AttributeResource
     */
    private $attributeResource;


    public function __construct(
        ModuleDataSetupInterface $moduleDataSetup,
        CustomerSetupFactory $customerSetupFactory,
        AttributeResource $attributeResource
    ) {
        $this->moduleDataSetup = $moduleDataSetup;
        $this->customerSetupFactory = $customerSetupFactory;
        $this->attributeResource = $attributeResource;
    }

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

    public function getAliases()
    {
        return [];
    }

    public static function getDependencies()
    {
        return [];
    }
}
```

Для того чтобы атрибут отображался в формах для него должны быть записи в таблице `customer_form_attribute`. Для формы в админке:
```php
$phoneAttribute = $eavSetup->getEavConfig()->getAttribute(
    \Magento\Customer\Model\Customer::ENTITY, 'phone'
);
$phoneAttribute->setData('used_in_forms', ['adminhtml_customer']);
$this->attributeResource->save($phoneAttribute);
```

Здесь сохранение через ресурс, т.к. репозитория с нужным ресурсом для атрибута нет, а метод `save()` в моделе устаревший.

## Describe how to extend the customer entity.

### How would you extend the customer entity using the extension attributes mechanism?

Пример использование extension_atributes в модуле `Magento_Newsletter`.

1. _etc/extension\_attributes.xml_
```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Api/etc/extension_attributes.xsd">
    <extension_attributes for="Magento\Customer\Api\Data\CustomerInterface">
        <attribute code="is_subscribed" type="boolean" >
            <join reference_table="newsletter_subscriber" reference_field="customer_id" join_on_field="entity_id">
                <field>subscriber_status</field>
            </join>
        </attribute>
    </extension_attributes>
</config>
```
2. Плагин в _etc/di.xml_
```xml
<type name="Magento\Customer\Api\CustomerRepositoryInterface">
    <plugin name="update_newsletter_subscription_on_customer_update"
            type="Magento\Newsletter\Model\Plugin\CustomerPlugin"/>
</type>
```
3. Класс плагина [`Magento\Newsletter\Model\Plugin\CustomerPlugin`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Newsletter/Model/Plugin/CustomerPlugin.php):
```php
public function afterGetById(CustomerRepositoryInterface $subject, CustomerInterface $customer)
{
    $extensionAttributes = $customer->getExtensionAttributes();
    if ($extensionAttributes === null || $extensionAttributes->getIsSubscribed() === null) {
        $isSubscribed = $this->getSubscriber($customer)->isSubscribed();
        $this->addIsSubscribedExtensionAttribute($customer, $isSubscribed);
    }
    return $customer;
}

private function addIsSubscribedExtensionAttribute(CustomerInterface $customer, bool $isSubscribed): void
{
    $extensionAttributes = $customer->getExtensionAttributes();
    if ($extensionAttributes === null) {
        /** @var CustomerExtensionInterface $extensionAttributes */
        $extensionAttributes = $this->extensionFactory->create(CustomerInterface::class);
        $customer->setExtensionAttributes($extensionAttributes);
    }
    $extensionAttributes->setIsSubscribed($isSubscribed);
}
```

## Describe how to customize the customer address.

### How would you add another field into the customer address?

[Customer Address Attributes (EE) (User guide)](https://docs.magento.com/user-guide/stores/attributes-customer-address.html)

В Enterprice Edition атрибуты можно редактировать из адинки.

В Community Edition атрибуты можно изменять/добавлять только программно с помощью [дата патчей](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/data-patches.html)

```php
<?php
namespace Oggetto\Exam\Setup\Patch\Data;

use Magento\Customer\Setup\CustomerSetup;
use Magento\Customer\Setup\CustomerSetupFactory;
use Magento\Framework\Setup\ModuleDataSetupInterface;
use Magento\Framework\Setup\Patch\DataPatchInterface;
use Magento\Eav\Api\AttributeRepositoryInterface;
use Magento\Customer\Model\ResourceModel\Attribute as AttributeResource;

class AddPhoneAttribute implements DataPatchInterface
{
    /**
     * @var ModuleDataSetupInterface
     */
    private $moduleDataSetup;
    /**
     * @var CustomerSetupFactory
     */
    private $customerSetupFactory;
    /**
     * @var AttributeResource
     */
    private $attributeResource;


    public function __construct(
        ModuleDataSetupInterface $moduleDataSetup,
        CustomerSetupFactory $customerSetupFactory,
        AttributeResource $attributeResource
    ) {
        $this->moduleDataSetup = $moduleDataSetup;
        $this->customerSetupFactory = $customerSetupFactory;
        $this->attributeResource = $attributeResource;
    }

    /**
     * @return AddPhoneAttribute|void
     * @throws \Magento\Framework\Exception\NoSuchEntityException
     * @throws \Magento\Framework\Exception\StateException
     */
    public function apply()
    {
        /** @var CustomerSetup $eavSetup */
        $eavSetup = $this->customerSetupFactory->create(['setup' => $this->moduleDataSetup]);

        $eavSetup->addAttribute('customer_address', 'customer_field', [
            'label' => 'Custom field',
            'input' => 'text',
            'type' => 'varchar',
            'sort_order' => 120,
            'position' => 120,
            'visible' => true,
            'user_defined' => true,
            'unique' => false,
            'required' => false,
            'system' => false,
        ]);

        $phoneAttribute = $eavSetup->getEavConfig()->getAttribute(
            'customer_field', 'phone'
        );
        $phoneAttribute->setData('used_in_forms', ['adminhtml_customer_address']);
        $this->attributeResource->save($phoneAttribute);
    }

    public function getAliases()
    {
        return [];
    }

    public static function getDependencies()
    {
        return [];
    }
}
```

Также как и с атрибутами кастомера, атрибуты адреса нужно добавлять в `customer_form_attribute` чтобы они отображались на формах в админке и на фронте.

[Add a new field in address form](https://devdocs.magento.com/guides/v2.4/howdoi/checkout/checkout_new_field.html)

## Describe customer groups and their role in different business processes.

[Customer Groups (User guide)](https://docs.magento.com/user-guide/customers/customer-groups.html)

### What is the role of customer groups?

Группы кастомеров используются для разделения кастомеров на группы.

Таблица БД: `customer_group`

Основные классы:

* [`Magento\Customer\Model\Data\Group`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/Data/Group.php) — модель, реализует [`Magento\Customer\Api\Data\GroupInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Api/Data/GroupInterface.php)
* [`Magento\Customer\Model\ResourceModel\GroupRepository`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/ResourceModel/GroupRepository.php) — репозиторий, реализует [`Magento\Customer\Api\GroupRepositoryInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Api/GroupRepositoryInterface.php)
* [`Magento\Customer\Model\GroupManagement`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/GroupManagement.php) — манеджер, реализует [`Magento\Customer\Api\GroupManagementInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Api/GroupManagementInterface.php), методы:
  * [`isReadonly()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/GroupManagement.php#L110) — проверяет предназначена ли группа только для чтения (такие группы нельзя удалить в админке и изменить их название). Такими группами является `NOT LOGGED IN` и дефолтные группы сторов
  * [`getDefaultGroup()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/GroupManagement.php#L124) — получить дефолтную группу для стора
  * [`getNotLoggedInGroup()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/GroupManagement.php#L150) — получить группу `NOT LOGGED IN`, группа предназначена для неавторизированных пользователей
  * [`getLoggedInGroups()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/GroupManagement.php#L158) — получить группы для авторизированных пользователей, это все кроме `NOT LOGGED IN` и `All customer group` (см. `getAllCustomersGroup()`)
  * [`getAllCustomersGroup()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/GroupManagement.php#L185) — возвращает `All customer group`, физически в базе её нет, метод возвращает объект группы с [константным ИД](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/GroupManagement.php#L32), использоуется для обозначения "для всех групп".
* [`Magento\Customer\Model\CustomerGroupConfig`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/CustomerGroupConfig.php) — модель конфигурации, реализует [`Magento\Customer\Api\CustomerGroupConfigInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Api/CustomerGroupConfigInterface.php), методы:
  * [`setDefaultCustomerGroup()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/CustomerGroupConfig.php#L39) — изменяет дефолтную группу в конфиге

### What functionality do they affect?

Группа может быть использована для:

* Применения различных налоговых классов
* Применение различных каталожный и корзиночных правил
* Применение различных цен продукта (tier price)

## Describe Magento functionality related to VAT.

[Value Added Tax (VAT) (User guide)](https://docs.magento.com/user-guide/tax/vat.html)

Основной модуль для функционала налогов: [`Magento_Tax`](https://github.com/magento/magento2/tree/2.4/app/code/Magento/Tax)

Для цены продукта есть дополнение [`tax`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Pricing/Adjustment.php), которое применяет налог к цене продукта, если это необходимо.

Для подсчёта налога используется класс [`Magento\Tax\Model\TaxCalculation`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Model/TaxCalculation.php) реализующий [`Magento\Tax\Api\TaxCalculationInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Api/TaxCalculationInterface.php). Подсчёт происходит в методе [`Magento\Tax\Model\TaxCalculation::calculateTax()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Model/TaxCalculation.php#L134), первым аргументом передаётся [`Magento\Tax\Model\Sales\Quote\QuoteDetails`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Model/Sales/Quote/QuoteDetails.php) реализующий [`Magento\Tax\Api\Data\QuoteDetailsInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Api/Data/QuoteDetailsInterface.php) для продуктов используется этот же объект, он создаётся в [`Magento\Catalog\Helper\Data::getTaxPrice()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Helper/Data.php#L590). Для подсчёта используются разные модели кулькуляторов наследуемые от [`Magento\Tax\Model\Calculation\AbstractCalculator`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Model/Calculation/AbstractCalculator.php), они создаётся в [`Magento\Tax\Model\Calculation\CalculatorFactory::create()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Model/Calculation/CalculatorFactory.php#L55). Калькуляторы реализуют методы:

* [`calculateWithTaxInPrice()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Model/Calculation/AbstractCalculator.php#L232) 
* [`calculateWithTaxNotInPrice()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Model/Calculation/AbstractCalculator.php#L242)

В обоих методах в результате возвращается [`Magento\Tax\Api\Data\TaxDetailsItemInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Api/Data/TaxDetailsItemInterface.php). После всех подсчётов `Magento\Tax\Model\TaxCalculation::calculateTax()` возвращет [`Magento\Tax\Api\Data\TaxDetailsInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Api/Data/TaxDetailsInterface.php) из которого можно получить информацию по налогам для квоты (или продукта).

Для кастомеров и продуктов можно создавать налоговые классы, они используются в налоговых правилах, которые используются для связи классов с налоговыми ставками (rate). В налоговых ставках указывается страна (и регион, если нужно) и процент налога.

### How do you customize VAT functionality?

При получении ставки налога есть ивент [`tax_rate_data_fetch`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Model/Calculation.php#L367), в котором предаётся запрос и отправитель: объект [`Magento\Tax\Model\Calculation`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Model/Calculation.php) котрому можно присвоить нужный процент налога с помощью метода `setRateValue()`.

Можно использовать плагины для метода [`Magento\Tax\Model\TaxCalculation::calculateTax()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Tax/Model/TaxCalculation.php#L134)
