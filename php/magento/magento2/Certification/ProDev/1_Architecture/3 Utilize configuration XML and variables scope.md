# Utilize configuration XML and variables scope

>Determine how to use configuration files in Magento. Which configuration files correspond to different features and functionality?

Конфигурации модулей M2, как и в M1, пишутся в `.xml` файлах. Помимо этого в M2 появились конфигурации деплоя, хранимые в `.php`.

## Конфигурации деплоя

[Magento's deployment configuration](https://devdocs.magento.com/guides/v2.4/config-guide/config/config-php.html)

Конфигураци деплоя содержат специфичные для текущего окружения конфигурации, по сути они заменят `etc/local.xml` из M1.

Конфигурации разделны между:

* `app/etc/config.php` — файл хранится в общем репозитории, содержит список модулей, тем, языковых пакетов и может содержать общие конфигурации.
* `app/etc/env.php` — файл находиться в гитигноре, содержит спецфичные для текущего инстанса параметры, например информация для конекта к БД.

Оба файла создаются при установке M2 и обязательны для её запуска.

В отличии от конфигураций модулей, конфигурации деплоя загружаются в память при инцилизазции M2, они не мерджатся с другими файлами и не могут быть исключены.

Сами файлы представляют из себя php файл который возвращает асоциатвный массив с параметрами (сохоже с тем как конфиги хранятся в ларавель).

Пример `env.php`:
```php
<?php
return [
    'backend' => [
        'frontName' => 'admin'
    ],
    'crypt' => [
        'key' => '017735685a97cf427dah88fb73e86461'
    ],
    'db' => [
        'table_prefix' => '',
        'connection' => [
            'default' => [
                'host' => 'localhost',
                'dbname' => 'magento2',
                'username' => 'magento',
                'password' => 'magento',
                'active' => '1'
            ]
        ]
    ],
    'resource' => [
        'default_setup' => [
            'connection' => 'default'
        ]
    ],
    'x-frame-options' => 'SAMEORIGIN',
    'MAGE_MODE' => 'developer',
    'session' => [
        'save' => 'files'
    ],
    'cache' => [
        'frontend' => [
            'default' => [
                'id_prefix' => '5a4_'
            ],
            'page_cache' => [
                'id_prefix' => '5a4_'
            ]
        ]
    ],
    'lock' => [
        'provider' => 'db',
        'config' => [
            'prefix' => NULL
        ]
    ],
    'cache_types' => [
        'config' => 1,
        'layout' => 1,
        'block_html' => 0,
        'collections' => 0,
        'reflection' => 0,
        'db_ddl' => 0,
        'compiled_config' => 0,
        'eav' => 0,
        'customer_notification' => 0,
        'config_integration' => 0,
        'config_integration_api' => 0,
        'full_page' => 0,
        'config_webservice' => 0,
        'translate' => 0,
        'vertex' => 0
    ],
    'install' => [
        'date' => 'Mon, 12 Aug 2019 18:21:32 +0000'
    ]
];

```

[`Magento\Framework\App\DeploymentConfig`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/DeploymentConfig.php) предоставляет доступ к конфигам.

### Управление установленными модулями

`config.php` содержит список установленных модулей:
```php
return array (
  'modules' =>
  array (
    'Magento_Core' => 1,
    'Magento_Store' => 1,
    'Magento_Theme' => 1,
    'Magento_Authorization' => 1,
    'Magento_Directory' => 1,
    'Magento_Backend' => 1,
    'Magento_Backup' => 1,
    'Magento_Eav' => 1,
    'Magento_Customer' => 1,
  ),
);

```
Соответственно, 1 — включен, 0 — выключен.

M2 позволяет управлять модулями через консоль или через графический интерфейс

* Удаление компонетов: [`bin/magento setup:uninstall`](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-uninstall.html)
* Включение/отключение компонетов: [`bin/magento module:disable`, `bin/magento module:enable`](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-enable.html#instgde-cli-subcommands-enable-disable)
* [Component Manager](https://devdocs.magento.com/guides/v2.4/comp-mgr/module-man/compman-start.html)
* [System Upgrade](https://devdocs.magento.com/guides/v2.4/comp-mgr/upgrader/upgrade-start.html)

### Приватные (sensitive) или спицифичные конфгурации

[Sensitive and environment settings](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/configuration/sensitive-and-environment-settings.html#how-to-specify-values-as-sensitive-or-system-specific)
[Sensitive and system-specific configuration settings](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/di-xml-file.html)

Для изменения приватных или специфичных для системы конфигураций используеться команда [`bin/magento config:sensitive:set`](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-config-mgmt-set.html).

Чтобы определеить параметр как приватный используеться [Magento\Config\Model\Config\TypePool](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Config/Model/Config/TypePool.php) референс в `di.xml` файлах:
```xml
<type name="Magento\Config\Model\Config\TypePool">
   <arguments>
      <argument name="sensitive" xsi:type="array">
         <item name="payment/test/password" xsi:type="string">1</item>
      </argument>
   </arguments>
</type>
```

## Конфигурации модулей

[Module configuration files](https://devdocs.magento.com/guides/v2.4/config-guide/config/config-files.html)

В M2 конфиги разбили на большее конличество файлов, файлы конфигурации подгружаются по требованию при небоходимости определённого типа конфигов. Можно определять кастомные типы конфигов.

Конфигурации разделяются по облостям (area):
* base (global) — глобальные конфиги, которые применяются ко всем областям
* frontend — пользовательская часть
* adminhtml — админка
* webapi_rest — REST API
* webapi_soap — SOAP API
* crontab — для cron джоб

Глоабльные конфиги хранятся в _`<module-dir>`/etc_, а area-специфичные в _`<module-dir>`/etc/`<area>`_

Файлы конфигурации одного типа мерджатся, area-специфичные конфиги перезаписывают глобальные.

Термены:
* *Configuration object* — маджетовсякая библиотека или класс для определения или валидации типа конфига, объект конфигурации для `config.xml` это [`Magento\Framework\App\Config`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Config.php)
* *Configuration stage* — стейджи определны как _primary_, _global_ и _arae_. Каждый стейдж определяет когда тип конфигурации загружается и мерджится.
* *Configuration scope* — так-же как и стейджи скопы определяют тип конфигурации. Например, `adminhtml` скопы загружаются вместе другими `adminhtml` конфигурациями модулей.

### Загрузка

Порядок загрузки:
* [`app/etc/di.xml`](https://github.com/magento/magento2/blob/2.4/app/etc/di.xml) используется для начальной загрузки M2.
* Глобальные конфигурации модулей _`<your component base dir>`/`<vendorname>`/`<component-type>`\_`<component-name>`/etc/*.xml_ сбоирает определённые файлы конфигов и мерджит их вместе.
* Area-специфичные конфигурации из модулей _`<your component base dir>`/`<vendorname>`/`<component-type>`\_`<component-name>`/etc/`<area>`/*.xml_ собирает конфигурации со всех модулей и мерджит вместе в глобальный конфиг, некторые area-специфичные конфиги могут переписывать глобальные или расширять их.
Где:
* `<your component base dir>` — базовая директория модуля, обычно _app/code_ или _vendor_
* `<vendorname>` — вендор
* `<component-type>` одно из: 
  * `module-` — модуль
  * `theme-` — тема
  * `language-` — языковой пакет
* `<component-name>` — имя компонента

### Мердж

Ноды в файлах конфигурации мерджаться на основе полного пути в xml, которые имеют специальный атрибут определённый в `$idAttributes` в качестве идентификатора. Идентификатор должен быть ункальным во всех вложеных нодах одного родителя.

Алгоритм мерджа:
* Если идентификаторы нод эквиваленты (или не определены) то весь вложенный контент презаписывается.
* Если идентфикаторы различны, то нода добавляется как дочерняя в родительскую
* Если исходный файл имеет несколько нод с одинаковыми именами, то возникает ошибка.

### Типы

[Configuration types and objects](https://devdocs.magento.com/guides/v2.4/config-guide/config/config-files.html)

Некоторые типы:
* module.xml — инициализационные конфиг моудля, содержит имя, версию (для не декларативной схемы) и soft зависимости
* events.xml — тут обработчики событий привязываются к событиям
* di.xml — конфигурация DI, здесь можно конфигурировать `preference`, `type`, `virtualType`, а так-же плагины
* config.xml — дефотлные значения для системных конфигов
* acl.xml — конфигурация ACL
* `<area>`/routers.xml — конфиги роутов
* adminhtml/system.xml — системные конфиги

### Интерфейсы

[Configuration interfaces](https://devdocs.magento.com/guides/v2.4/config-guide/config/config-files.html)

Взаимодействовать с конфигами можно с помощью интерфейсов [Magento\Framework\Config](https://github.com/magento/magento2/tree/2.4/lib/internal/Magento/Framework/Config). Так-же их можно использовать для создания своего типа конфига.

## Создание типа

[Create or extend configuration types](https://devdocs.magento.com/guides/v2.4/config-guide/config/config-create.html)

Для создания необходимо:
* XML файлы конфигурации
* XSD для валидации
* Загрузчик

1. Cоздать файл XSD для валидации
2. Создать файл XML для конфигурации
3. Дбоавить в `di.xml`:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">

    <type name="Magento\Sales\Model\Order\Pdf\Config\Reader">
        <arguments>
            <argument name="fileName" xsi:type="string">pdf.xml</argument>
            <argument name="converter" xsi:type="object">Magento\Sales\Model\Order\Pdf\Config\Converter</argument>
            <argument name="schemaLocator" xsi:type="object">Magento\Sales\Model\Order\Pdf\Config\SchemaLocator</argument>
        </arguments>
    </type>

    <virtualType name="pdfConfigDataStorage" type="Magento\Framework\Config\Data">
        <arguments>
            <argument name="reader" xsi:type="object">Magento\Sales\Model\Order\Pdf\Config\Reader</argument>
            <argument name="cacheId" xsi:type="string">sales_pdf_config</argument>
        </arguments>
    </virtualType>

    <type name="Magento\Sales\Model\Order\Pdf\Config">
        <arguments>
            <argument name="dataStorage" xsi:type="object">pdfConfigDataStorage</argument>
        </arguments>
    </type>
</config>
```
4. Создать класс ридер имплементирующий [`Magento\Framework\Config\ReaderInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Config/ReaderInterface.php), можно унаследоваться от [`Magento\Framework\Config\Reader\Filesystem`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Config/Reader/Filesystem.php) у него нужно использовать эти параметры (в конструкторе)
* `$fileResolver`. Реализует [`\Magento\Framework\Config\FileResolverInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Config/FileResolverInterface.php) читает файлы конфигов
* `$converter` реализует [`\Magento\Framework\Config\ConverterInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Config/ConverterInterface.php). Конвертирует xml в массив
* `$schemaLocator` реализует [`\Magento\Framework\Config\SchemaLocatorInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Config/SchemaLocatorInterface.php) предоставляет полные пути для валидации файлов
* `$validationState` реализует [`\Magento\Framework\Config\ValidationStateInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Config/ValidationStateInterface.php). Определяет валидацию файла
* `$fileName` имя файла конфигурации, ридер будет искать файлы с таким именем в _etc_ директории модулей.
* $idAttributes. список идентификаторов нод
```
array(
    '</path/to/node>' => '<identifierAttributeName>',
    '</path/to/other/node>' => '<identifierAttributeName>',
}
```
* `$defaultScope` определяет какой скоп должен читаться по дефолту, по дефотлту стоит `global`.

Пример: [`Magento\Sales\Model\Order\Pdf\Config`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Sales/Model/Order/Pdf/Config.php)

### Валидация

[Validate a configuration type](https://devdocs.magento.com/guides/v2.4/config-guide/config/config-create.html)

В M2 файлы конфигов могут валидироваться с помощью XSD схем, содержащих описания. XSD файл должен иметь такое же имя, как и файл конфигурации. Можно делать xsd как для оригинальных файлов так и для смердженого.

Например:
* `events.xml` — файл конфигурации
* `events.xsd` — файл для валидации
* `events_merged.xsd` — файл для валидации смердженных конфигов

XSD подключается с помощью URN:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager:etc/config.xsd">
```
