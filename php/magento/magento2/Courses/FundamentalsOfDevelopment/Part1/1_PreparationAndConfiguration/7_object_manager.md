# Менеджер объектов

[ObjectManager](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/ObjectManager/ObjectManager.php)

В M2 менеджер объектов заменяет `Mage` класс из M1, он создаёт все объекты и отвечает за всё, что с этим связано. Менеджер объектов имеет два метода:

* `get` — создаёт и возвращает singleton инстанс объекта
```php
    public function get($type)
    {
        $type = ltrim($type, '\\');
        $type = $this->_config->getPreference($type);
        if (!isset($this->_sharedInstances[$type])) {
            $this->_sharedInstances[$type] = $this->_factory->create($type);
        }
        return $this->_sharedInstances[$type];
    }
```
* `create` — создаёт новый инстанс объекта
```php
    public function create($type, array $arguments = [])
    {
        return $this->_factory->create($this->_config->getPreference($type), $arguments);
    }
```

Менеджер объектов связан с передаваемыми в конструкторе параметрами:

* он парсит передаваемые типы параметров и имя переменных
* для определения какие аргументы должны быть преданы для параметров используется конфигурация
* он создаёт нужные аргументы рекрсивно

## Использование

Обычно, лучше избегать использование менеджера объектов напрямую, вместо этого нужно использовать DI и запрашивать все нужные классы/интерфейсы в конструкторе.

## Автоматически-генерируемые классы

* Некоторые классы в M2 генерируются автоматически, например:
    * Фабрики — классы для получения инстансов объектов-сущностей, таких как продукты
    * intercepter — классы для работы плагинов, M2 генерирует их автоматически, при наличии плагина для класса, intercepter создаётся с теми же методами, что в классе, но добавляет к ним вызов плагинов. Для работы с плагинами используется [треит Interceptor](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Interception/Interceptor.php)
    * proxy — инструмент помогающий со сложными завсимостями, нужен для работы DI. Напрямую не используется.
* Сгенерированные классы хранятся в директории `generated`.

## Конфигурация

Для определения какие зависимости необходимо предоставить менеджер объектов использует конфигурацию, она распологается в:

* _app/etc/di.xml_ — общая конфигурация, содержит настройки для ключевых классов, необходимых для работы системы
* _`<module_dir>`/etc/di.xml_ — модульная конфигурация 
* _`<module_dir>`/etc/`<area>`/di.xml_ — конфигурация для определённой области

Во время запроса система работает с мердженой конфгурацией.

### Preferences

Preferences определяет какой класс будет инстанцировать аргумент в конструкторе, preferences можно названачать как для интерфейсов, так и для классов, но предпочтительней использовать именно интерфейсы.

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <preference for="Magento\Catalog\Api\Data\ProductInterface" type="Magento\Catalog\Model\Product" />
</config>
```

теперь при вызове в конструкторе аргумента с типом `Magento\Catalog\Api\Data\ProductInterface` в переменную будет инстанцироватся класс `Magento\Catalog\Model\Product`, этот конфиг находится в `Magento/Catalog/etc/di.xml`.

### Arguments

Arguments преднозначено для передачи агументов объектам, которые инстунцируются в конструкторе. Аргументы могут быть разлчных типов: `string`, `array`, `object` и т.д., DI позволяет позволяет определить какие объекты должны инстнцироваться при создании нового инстанса.

```xml
<type name="Magento\Catalog\Helper\Product">
    <arguments>
        <argument name="catalogSession" xsi:type="object">Magento\Catalog\Model\Session\Proxy</argument>
        <argument name="reindexPriceIndexerData" xsi:type="array">
            <item name="byDataResult" xsi:type="array">
                <item name="tier_price_changed" xsi:type="string">tier_price_changed</item>
            </item>
            <item name="byDataChange" xsi:type="array">
                <item name="status" xsi:type="string">status</item>
                <item name="price" xsi:type="string">price</item>
                <item name="special_price" xsi:type="string">special_price</item>
                <item name="special_from_date" xsi:type="string">special_from_date</item>
                <item name="special_to_date" xsi:type="string">special_to_date</item>
                <item name="website_ids" xsi:type="string">website_ids</item>
                <item name="gift_wrapping_price" xsi:type="string">gift_wrapping_price</item>
                <item name="tax_class_id" xsi:type="string">tax_class_id</item>
            </item>
        </argument>
        <argument name="reindexProductCategoryIndexerData" xsi:type="array">
            <item name="byDataChange" xsi:type="array">
                <item name="category_ids" xsi:type="string">category_ids</item>
                <item name="entity_id" xsi:type="string">entity_id</item>
                <item name="store_id" xsi:type="string">store_id</item>
                <item name="website_ids" xsi:type="string">website_ids</item>
                <item name="visibility" xsi:type="string">visibility</item>
                <item name="status" xsi:type="string">status</item>
            </item>
        </argument>
        <argument name="productRepository" xsi:type="object">Magento\Catalog\Api\ProductRepositoryInterface\Proxy</argument>
    </arguments>
</type>
```

В некоторых случаях менеджер объектов не может предоставить нужный инстанс, напрмер, когда нужен определённый экземпляр продукта, менеджер может только предоставить общий объект этого класса, но не может загружать конкретные сущности. Для загрузки сущностей используются репозитории.

По умолчанию объекты в аргументах являются синглтонами, если нужено использовать не синглтон, то нужно указывать в параметре `shared` "false".

```xml
<type name="Magento\Catalog\Model\Indexer\Product\Price\Processor">
    <arguments>
        <argument name="indexer" xsi:type="object" shared="false">Magento\Framework\Indexer\IndexerInterface</argument>
    </arguments>
</type>
```

### VirtualType

Виртуальные типы используются для внедрения зависимостей в существующие PHP-классы, без влияния на другие классы и без необходимости создавать новые. Виртуальный тип может быть использован только в конфигурации `di.xml`. Он не может быть расширен и не может использоваться как заивисимость в конструкторе. Они позволяют изменить аргументы специфичных внедряемых зависимостей, тем самым изменяя поведения конкретных классов:

```xml
<virtualType name="Magento\Catalog\Pricing\Price\Pool" type="Magento\Framework\Pricing\Price\Pool">
    <arguments>
        <argument name="prices" xsi:type="array">
            <item name="regular_price" xsi:type="string">Magento\Catalog\Pricing\Price\RegularPrice</item>
            <item name="final_price" xsi:type="string">Magento\Catalog\Pricing\Price\FinalPrice</item>
            <item name="tier_price" xsi:type="string">Magento\Catalog\Pricing\Price\TierPrice</item>
            <item name="special_price" xsi:type="string">Magento\Catalog\Pricing\Price\SpecialPrice</item>
            <item name="base_price" xsi:type="string">Magento\Catalog\Pricing\Price\BasePrice</item>
            <item name="custom_option_price" xsi:type="string">Magento\Catalog\Pricing\Price\CustomOptionPrice</item>
            <item name="configured_price" xsi:type="string">Magento\Catalog\Pricing\Price\ConfiguredPrice</item>
            <item name="configured_regular_price" xsi:type="string">Magento\Catalog\Pricing\Price\ConfiguredRegularPrice</item>
        </argument>
    </arguments>
</virtualType>
```
