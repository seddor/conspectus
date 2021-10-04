# Demonstrate the ability to manage the cache

>Describe cache types and the tools used to manage caches. How do you add dynamic content to pages served from the full page cache?

>Describe how to operate with cache clearing. How would you clean the cache? In which case would you refresh cache/flash cache storage?

>Describe how to clear the cache programmatically. What mechanisms are available for clearing all or part of the cache?

[Magento cache overview](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/cache_for_frontdevs.html)

M2 поддерживает такие способы хранения кешей:

* Файлы (по умолчанию)
* [БД](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/cache/partial-caching/database-caching.html)
* [Redis](https://devdocs.magento.com/guides/v2.4/config-guide/redis/redis-pg-cache.html)
* [Varnish](https://devdocs.magento.com/guides/v2.4/config-guide/varnish/config-varnish.html)

## Публичный и приватный контент

### Публичный

[Public content](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/cache/page-caching/public-content.html) — хранится на сервере и доступен для нескольких пользователей.

Кешируются только `GET` и `HEAD` методы.

В M2 по умолчаню все страницы кешруемые, если блок в лаяуте некешруемый, то страница тоже становится некешируемой. Для создания некешируемой страницы нужно в блоке лаяута указать `cacheable="false"`:
```xml
<layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/layout_generic.xsd">
    <container name="root">
        <block class="Magento\Paypal\Block\Payflow\Link\Iframe" name="payflow.link.iframe" template="Magento_Paypal::payflowlink/redirect.phtml" cacheable="false"/>
    </container>
</layout>
```

#### Определение политики кеша

Можно определить на стороне веб-сервеса или в коде, через контроллер:
```php
class DynamicController extends \Magento\Framework\App\Action\Action
{
    protected $pageFactory;

    public function __construct(
        \Magento\Framework\App\Action\Context $context,
        \Magento\Framework\View\Result\PageFactory $resultPageFactory
    ) {
        parent::__construct($context);
        $this->pageFactory = $resultPageFactory;
    }

    /**
     * This action render random number for each request
     */
    public function execute()
    {
        $page = $this->pageFactory->create();
        //We are using HTTP headers to control various page caches (varnish, fastly, built-in php cache)
        $page->setHeader('Cache-Control', 'no-store, no-cache, must-revalidate, max-age=0', true);

        return $page;
    }
}
```

#### Настройка вариаций

M2 для уникального ключа кеша использует HTTP контекстные переменные, получить ключ можно методом [`Magento\Framework\App\Http\Context::getVaryString()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Http/Context.php#L111). 

Пример разделения кеша по возросту пользователей(для тех кому есть 18, и нет):
```php
/**
 * Plugin on \Magento\Framework\App\Http\Context
 */
class CustomerAgeContextPlugin
{
    public function __construct(
        \Magento\Customer\Model\Session $customerSession
    ) {
        $this->customerSession = $customerSession;
    }
    /**
     * \Magento\Framework\App\Http\Context::getVaryString is used by Magento to retrieve unique identifier for selected context,
     * so this is a best place to declare custom context variables
     */
    function beforeGetVaryString(\Magento\Framework\App\Http\Context $subject)
    {
        $age = $this->customerSession->getCustomerData()->getCustomAttribute('age');
        $defaultAgeContext = 0;
        $ageContext = $age >= 18 ? 1 : $defaultAgeContext;
        $subject->setValue('CONTEXT_AGE', $ageContext, $defaultAgeContext);
    }
}
```

#### Кука `X-Magento-Vary`

Для перемещения контекста меджу HTTP слоями использоуется кука `X-Magento-Vary`

#### Инвалидация 

Можно чистить кеш сразу после изменения сущности, для этого нужно чтобы класс реализовывал [`Magento/Framework/DataObject/IdentityInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/DataObject/IdentityInterface.php)

1. Для модели сущности
```php
class Product implements Magento\Framework\DataObject\IdentityInterface;
{
     /**
      * Product cache tag
      */
     const CACHE_TAG = 'catalog_product';
    /**
     * Get identities
     *
     * @return array
     */
    public function getIdentities()
    {
         return [self::CACHE_TAG . '_' . $this->getId()];
    }
}
```
2. Для блока:
```php
class View extends AbstractProduct 
    implements \Magento\Framework\DataObject\IdentityInterface
{
    /**
     * Return identifiers for produced content
     *
     * @return array
     */
    public function getIdentities()
    {
        return $this->getProduct()->getIdentities();
    }
}
```

### Приватный

[Private content](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/cache/page-caching/private-content.html) — хранится на стороне клиента (в браузере) и специфичен для каждого пользователя.

Для приватного контента используется JS либа [`customer-data`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/view/frontend/web/js/customer-data.js). Данные хранятся в локал сторе браузера.

#### Создание источника

Для передачи данных с бекенда нужно:

1. Cоздать класс в _`<module-dir>`/CustomerData_, класс должен реализоывать [`Magento\Customer\CustomerData\SectionSourceInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/CustomerData/SectionSourceInterface.php).

[Пример](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/CustomerData/CompareProducts.php#L62-L69)

2. добавить информацию в `di.xml`:
```xml
<type name="Magento\Customer\CustomerData\SectionPoolInterface">
    <arguments>
        <argument name="sectionSourceMap" xsi:type="array">
            <item name="custom-name" xsi:type="string">Vendor\ModuleName\CustomerData\ClassName</item>
        </argument>
    </arguments>
</type>
```

#### Создание блока и темлейта

1. В шаблоне размещаются плейсхолдеры, используется синтаксис [Knockout](https://knockoutjs.com/documentation/introduction.html), в корневом элементе указывается скоуп, например: `data-bind="scope: 'compareProducts'"`
[Пример шаблона](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/view/frontend/templates/product/compare/sidebar.phtml#L46-L48)
2. Инцилизируется компонент:
```html
<script type="text/x-magento-init">
    {"<css-selector>": {"Magento_Ui/js/core/app": <?php echo $block->getJsLayout();?>}}
</script>
```

#### Конфигурация UI компонента

UI-компонент рендерит данные блока на фронт, для инициализации компонента вызвается метод `_super()`.
[Пример](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/view/frontend/web/js/view/compare-products.js)
[Пример определения в лаяуте](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/view/frontend/layout/default.xml#L11-L22)

#### Инвалидация

Экшены ивалидирующие приватные данные конфигурируются в _`<module-dir>`/etc/frontend/sections.xml_, также M2 инвалидирует кеш на POST и PUT запросы.
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Customer:etc/sections.xsd">
    <!-- invalidates the "compare-products" section when a user
    adds a product to the comparison, resulting in a "catalog/product_compare/add" POST request -->
    <action name="catalog/product_compare/add">
        <section name="compare-products"/>
    </action>
    <!-- invalidates the section when a customer removes a product from the comparison -->
    <action name="catalog/product_compare/remove">
        <section name="compare-products"/>
    </action>
    <!-- invalidates the section when a customer clears all products from the comparison -->
    <action name="catalog/product_compare/clear">
        <section name="compare-products"/>
    </action>
</config>
```

## Статичные файлы

M2 генерирует статичные файлы, их можно очистить так:

* Через админку
* Руками:
```bash
rm -rf pub/static/*
rm -rf var/view_preprocessed/*
```
* некоторые команды поддерживают опцию `--clear-static-content`:
```bash
bin/magento module:enable
bin/magento module:disable
bin/magento theme:uninstall
bin/magento module:uninstall
```

## Типы кешей

[Manage the cache](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-cache.html)

* `config` — смердженные конфигурации всех модулей, так-же содержит стор-специфичные настройки хранимые в системных файлах и ДБ. Нужно чистить после изменения конфигов.
* `layout` — скомпилированные лаяуты со всех компонентов. Чистить после измения файлов лаяутов
* `block_html` — кеш HTML блоков. Чистить после измения всего, что относиться ко вью (блоки, лаяуты)
* `collections` — результаты запросов в БД, M2 чистит их при необходимости, но сюда могут писаться данные модулей сторонних разработчиков. Нужно чистить если кастомный модуль пишет в этот кеш и он не чиститься при необходимости.
* `db_ddl` — схема ДБ. Чистится автоматически, может использоваться сторонки разработчиками. Единственный способ обновить схему автоматически: команда `magento setup:db-schema:upgrade`.
* `compiled_config` — скомпилированные конфигурации
* `eav` — метадата для EAV атрибутов, обычно не нужно чистить. 
* `full_page` — сгенерированные HTML-страницы, чистится автоматически, может использоваться сторонки разработчиками. 
* `reflection` —  убирает зависимость меджу Webapi модулем и Customer модулем
* `translate` — переводы смердженные со всех модулей 
* `config_integration`
* `config_integration_api`
* `config_webservice` — структура Web API
* `customer_notification` — временные уведомления пользовательского интерфейса

## Просмотр статуса кеша

```bash
bin/magento cache:status
```

## Включение/отключение типов кеша

* включение
```bash
bin/magento cache:enable [type] ... [type]
```
* отключение
```bash
bin/magento cache:disable [type] ... [type]
```

## Очистка и сброс кеша

* Очистка удаляет элементы из кеша, а итоге будут удалены только элементы в кеше которые использует M2.
```bash
bin/magento cache:clean [type] ... [type]
```
* Сброс полностью удаляет все элементы из хранилища кеша, т.е. хранилище полностью очищается.
```bash
bin/magento cache:flush [type] ... [type]
```
