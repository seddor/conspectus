# Demonstrate how to use dependency injection

>Describe Magento’s dependency injection approach and architecture. How are objects realized in Magento? Why is it important to have a centralized process creating object instances?

>Identify how to use DI configuration files for customizing Magento. How can you override a native class, inject your class into another object, and use other techniques available in `di.xml` (such as `virtualTypes`)?

[О паттерне](https://designpatternsphp.readthedocs.io/ru/latest/Structural/DependencyInjection/README.html)
[Dependency injection](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/depend-inj.html)

M2 использует внедрение зависимостей в конструкторе, когда все зависимости объекта или фабрики предоставляются при создании класса.

DI в M2 призван заменть функциональность класса `Mage` из M1. 

## Принцип инверсии зависимостей

Все классы высокого уровня должны рабоать с интерфейсами вместо классов.

В M2 для указания конкретного класса для интерфейса служат файлы `di.xml`

## di.xml

[The di.xml file](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/di-xml-file.html)

Содержит конфигурации для зависимостей, которые будут внедрены менеджером объектов, также здесь определяются приватные конфигуации.

### Areas и точки входа

Каждый модуль имеет глобальный `di.xml` и area-специфичный, M2 читает все файлы и мерджит вместе. 

Порядок загрузки такой:
* Начальный (_app/etc/di.xml_)
* Глобальный (_`<moduleDir>`/etc/di.xml_)
* Area-специфичный (_`<moduleDir>`/etc/`<area>`/di.xml_)

### Тип конфигурации

Тип конфигурации указывает жизненый цикл объекта и как его инстанцировать.

Пример конфигурации типа:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <virtualType name="moduleConfig" type="Magento\Core\Model\Config">
        <arguments>
            <argument name="type" xsi:type="string">system</argument>
        </arguments>
    </virtualType>
    <type name="Magento\Core\Model\App">
        <arguments>
            <argument name="config" xsi:type="object">moduleConfig</argument>
        </arguments>
    </type>
</config>
```
здесь:
* `moduleConfig` — виртуальный тип расширяющий `Magento\Core\Model\Config`
* `Magento\Core\Model\App` — все инстансы типа `Magento\Core\Model\App` инстанцируют `moduleConfig` как заивисимость.

### Виртуальные типы

Виртуальные типы используются для внедрения зависимостей в существующие PHP-классы, без влияния на другие классы и без необходимости создавать новые. Вртуальный тип может быть использован только в конфигурации `di.xml`. Он не может быть расширен и не может использоваться как заивисимость в конструкторе. Они позволяют изменить аргументы специфичных внедряемых зависимостей, тем самым изменяя поведения конкретных классов.

Пример:
```xml
<type name="Oggetto\SkinCareDiagnosticMindbox\Model\ValueMapRepository">
    <arguments>
        <argument name="valueMaps" xsi:type="array">
            <item name="skin_type" xsi:type="object">Oggetto\SkinCareDiagnosticMindbox\Model\ValueMap\SkinType</item>
        </argument>
    </arguments>
</type>
<virtualType
        name="Oggetto\SkinCareDiagnosticMindbox\Model\ValueMap\SkinType"
        type="Oggetto\SkinCareDiagnosticMindbox\Model\ValueMap">
    <arguments>
        <argument name="map" xsi:type="array">
            <item name="combination" xsi:type="object">Oggetto\SkinCareDiagnosticMindbox\Model\ValueMap\SkinType\Combination</item>
        </argument>
    </arguments>
</virtualType>
```
Здесь:
* `Oggetto\SkinCareDiagnosticMindbox\Model\ValueMapRepository` — реальный класс
* `Oggetto\SkinCareDiagnosticMindbox\Model\ValueMap\SkinType` — виртуальный тип, физически этого класса не существует, при его создании создастся инстанст класса `Oggetto\SkinCareDiagnosticMindbox\Model\ValueMap`, которому в качестве аргуметов конструктора будут переданы параметры указаные в конфигурации виртуального типа, т.е. по сути виртуальные типа это заранее сконфигурированные классы.


### Аргументы конструктора

Аргументы конструктора меняются в ноде `arguments`. Объектный менеджер внедряет зависимости в класс при создании. Имена заданые в конфигах должны соответствовать именам аргументов в конструкторе.

Пример создания инстанса `Magento\Core\Model\Session` вместе с аргументом `$sessionName` со значеием `adminhtml`:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <type name="Magento\Core\Model\Session">
        <arguments>
            <argument name="sessionName" xsi:type="string">adminhtml</argument>
        </arguments>
    </type>
</config>
```

#### Типы аргуметов

##### object

```xml
<argument xsi:type="object">{typeName}</argument>
<argument xsi:type="object" shared="{shared}">{typeName}</argument>
```

Создаёт инстанс `{typeName}` и передаёт как аргумент, параметр `shared` определяет жизненный цикл объекта.

Жизненный цикл определяет сколько инстансов объекта может существовать. У объекта могут быть такие жизненные циклы:

* **Singleton** — по умолчанию, создаётся один инстанс класса и возвращается всегда. (`shared="true"`)
* **Transient** — создаётся новый инстанс на каждый запрос. (`shared="false"`)

`shared` можно устанавливать как для типов так и для аргуметов.

##### string

```xml
<argument xsi:type="string" translate="true">{strValue}</argument>
```

##### boolean

```xml
<argument xsi:type="boolean">{boolValue}</argument>
```

Данные будут сконвертированы в булевое значение, в соответствии с таблицей ниже:

|InputType|Data|Boolean Value|
|-|-|-|
|Boolean|true|true|
|Boolean|false|false|
|String|"true"|true|
|String|"false"|false|
|String|"1"|true|
|String|"0"|false|
|Integer|1|true|
|Integer|0|false|

##### number

```xml
<argument xsi:type="number">{numericValue}</argument>
```

все что является числом и [numeric strings](https://www.php.net/is_numeric)

##### init_parameter или const

```xml
<argument xsi:type="init_parameter">{Constant::NAME}</argument>
<argument name="scopeType" xsi:type="const">Magento\Store\Model\ScopeInterface::SCOPE_STORE</argument>
```

константы PHP классов

##### null

```xml
<argument xsi:type="null"/>
```

##### array

```xml
<argument xsi:type="array">
  <item name="someKey" xsi:type="<type>">someVal</item>
</argument>
```

Массив может содержать элементы любых типов, в том числе вложенные массивы. При мердже конфигов, массивы с одинаковыми ключами марджатся в новый массив. Массивы из боллее поздних по загузке конфгов переопределяют прошлый.

### Abstraction-implementation mappings

Используеться менеджером объектов когда конструктор класса запрашивает объект через интерфейс. Объектный менеджер используется для определения конкретной реализации для класса в текущем скопе.

```xml
<!--  File: app/etc/di.xml -->
<config>
    <preference for="Magento\Core\Model\UrlInterface" type="Magento\Core\Model\Url" />
</config>
```
Таким образом при внедрении зависимости `Magento\Core\Model\UrlInterface` будет использован класс `Magento\Core\Model\Url`.

Зависимость можно переопределить:
```xml
<!-- File: app/code/Magento/Backend/etc/adminhtml/di.xml -->
<config>
    <preference for="Magento\Core\Model\UrlInterface" type="Magento\Backend\Model\Url" />
</config>
```
Теперь в area `admin` при внедрении `Magento\Core\Model\UrlInterface` будет использоваться `Magento\Backend\Model\Url`.

### Иерархия параметров конфигурации

Параметры сконфигурированные для типа класса, также применяются для его потомков, при необходисомти параметры для потомков можно переопределить.

### Получения информации о зависимостях класса

Используется команда [`dev:di:info`](https://devdocs.magento.com/guides/v2.4/reference/cli/magento.html#devdiinfo), которая возвращает всю нформацию о параметрах конструктов, плагинах и прочих зависимостях класса.

## Менеджер объектов

[ObjectManager](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/object-manager.html)

Объект используемый в M2 для инстанцированния объектов, реализует [`Magento\Framework\ObjectManagerInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/ObjectManagerInterface.php) 

В обязаности объектоного менеджера входит:

* Создание объектов в фабриках и проксях
* Реализация синглотона
* Управление зависимостями через конструкторы классов
* Автоматическое инстанцирования параметров в конктрукторы классов

Объектный менеджер настраивается через `di.xml`. 

### Использование

M2 использует объектный менеджер для генерированния классов и внедрения зависимостей, классы не должны вызывать объектный менеджер напрямую. Напрямую объектный менеджер используеться только в [фабриках](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/factories.html). Создание объектов допускается через [фабрики](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/factories.html) или [прокси](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/proxies.html) обычно их генерирует сама M2.

### Исключения

В некоторых случаях допустимо использования объектного менеджаера:

* Можно использовать в магических метода `__wakeup()`, `__sleep()`, etc. Например как [здесь](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/Entity/Attribute/AbstractAttribute.php#L1434);
* При необходимости обеспечить обратную совместимость для конструктора;
* В глобальном скопе можно использовать для фикстур интеграционных тестов;
* Может быть зависимостью для фабрик или проксей.

#### Фабрики

[Factories](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/factories.html)

Используеются для инстанцирования не инициизируемых зависимостей. Создаются автоматически при запросе `<название-класса>`Factory, например: класса `Oggetto\SkinCareDiagnostic\Api\Data\ResultInterfaceFactory` нет, однако при его запросе в конструкторе, маджента автоматически создаст класс с таким содержимым:
```php
class ResultInterfaceFactory
{
    /**
     * Object Manager instance
     *
     * @var \Magento\Framework\ObjectManagerInterface
     */
    protected $_objectManager = null;

    /**
     * Instance name to create
     *
     * @var string
     */
    protected $_instanceName = null;

    /**
     * Factory constructor
     *
     * @param \Magento\Framework\ObjectManagerInterface $objectManager
     * @param string $instanceName
     */
    public function __construct(\Magento\Framework\ObjectManagerInterface $objectManager, $instanceName = '\\Oggetto\\SkinCareDiagnostic\\Api\\Data\\ResultInterface')
    {
        $this->_objectManager = $objectManager;
        $this->_instanceName = $instanceName;
    }

    /**
     * Create class instance with specified parameters
     *
     * @param array $data
     * @return \Oggetto\SkinCareDiagnostic\Api\Data\ResultInterface
     */
    public function create(array $data = [])
    {
        return $this->_objectManager->create($this->_instanceName, $data);
    }
}
```
Это класс является фабрикой и его нужно использовать для создания новых объектов класса `Oggetto\SkinCareDiagnostic\Api\Data\ResultInterface`.

#### Прокси

Используются для управлениеями зависимостями класса, например для ленивой инициализации объектов. В основном используется для повышения производтельности для тяжёлых классов (с большим количеством зависимостей), прокси инициализирует зависимости лениво, а не сразу.

[Proxies](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/proxies.html)
[О паттерне (refactoring.guru)](https://refactoring.guru/ru/design-patterns/proxy)
[О паттерне (PHP)](https://designpatternsphp.readthedocs.io/ru/latest/Structural/Proxy/README.html)
[Proxies](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/proxies.html)
[Magento 2 Object Manager: Proxy Objects](https://alanstorm.com/magento_2_object_manager_proxy_objects/)

## Типы внедрения

### Constructor injection

```php
    public function __construct(
        Magento\Backend\Model\Menu\Item\Factory $menuItemFactory,  // Service dependency
    ) {
        $this->_itemFactory = $menuItemFactory;
    }
```

определяются в конструкторе классов и настраиваются в `di.xml`.

### Method injection

```php
    public function processCommand(\Magento\Backend\Model\Menu\Builder\AbstractCommand $command) // API param
    {
        // processCommand Code
    }
```

Внедрение в метод использует параметр метода, используется когда объект должен выполнить действие с заисимостью, которую нельзя внедрить в конструктор или её внедрение несёт сильные затраты.

## Типы зависимостей

### Инициируемые (Injectable)

Синголтонны сервисные объекты полученые при внедрении зависимостей. Внедряются с использованием конфига из `di.xml`. Могут зависеть от других инициируемых объектов в своём конструкторе.

### Не иницируемые (Newable/non-injectable)

Объекты, которые создают новые инстансы каждый раз, когда они нужны. Времменные объекты, требующие внешнего ввода от пользователя или БД, тоже попадают в этот тип. 

Напрмер, нельзя заивисеть от класса продукта, вместо этого нужно использовать фабрику, которая сможет произовдить объекты продуктов.

## Подробнее

[Alan Storm, The Magento 2 Object System](https://alanstorm.com/category/magento-2/#magento-2-object-system)
