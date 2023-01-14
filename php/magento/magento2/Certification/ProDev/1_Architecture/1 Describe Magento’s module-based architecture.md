# Describe Magento’s module-based architecture

>Describe module limitations. How do different modules interact with each other? What side effects can come from this interaction?

M2, как и M1 является модульной системой.

Модуль — это логическая группа содержащая блоки, контроллеры, хелперы, модели — релевантные какой-либо бизнес фиче. В иделае каждый модуль должен инкасулировать одну фичу и иметь минимальные зависимости от других. 

Модули (а также темы) используются для кастомизации в M2. Модуль предоставлет бизнес фичи с соответствующей логикой, темы используются для изменения интерфейса (User Experience) и изменения его внешенего вида. Модули и темы имеют жизнненый цикл, их можно устанавливать, удалять и отключать. 

### Местоположение

Модули добавляемые через composer (в том числе базовые модули маджетны) находяться по пути _vendor/`<vendor>`/`<type>_<module_name>`_

Здесь `<type>` может быть:

* **module** — для модулей (`module-catalog`)
* **theme** — для фронтенд/админ тем (`theme-frontend-luma`)
* **language** — для языковых пакетов (`language-en_us`)

Специфичные для проекта модули можно добавлять по пути _app/code/`<vendor>`/`<type>_<module-name>`_

### Создание модуля

[Create a New Module](https://devdocs.magento.com/videos/fundamentals/create-a-new-module/)
[Package a component](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/package/package_module.html)

1. Создать директорию модуля

Для специфичных для проекта модулей используеться директория _app/code_. 

Общие модули нужно добавлять через composer, поэтому для такого модуля необходимо создать `composer.json` файл.

Пример содержимого:
```json
{
    "name": "<venodr>/<module-name>",
    "description": "Magento 2 module sample",
    "require": {
        "php": "~7.2.0",
        "magento/framework": "102.0.*",
    },
    "type": "magento2-module",
    "license": [
        "proprietary"
    ],
    "autoload": {
        "files": [
            "registration.php"
        ],
        "psr-4": {
            "<vendor>\\<module-name>\\": ""
        }
    }
}
```

* **name** — имя модуля, все символы должны быть в нижнем регистре, для разделения используеться `-`.
* **type** — тип компонента, для M2 могут быть такие значения:
    * **metapackage** — метапакет предназначен только для технического использования, содержит только `composer.json` файл с перечисление необходимых пакетов. Например Magento Open Source и Magento Commerce являються метапакетами.
    * **magento2-module** — модуль
    * **magento2-theme** — тема
    * **magento2-language** — языковой пакет
    * **magento2-library** — библиотеки распологаемые в _lib/internal_ вместо _vendor_
    * **magento2-component** — пакет сформированный из файлов, которые должны располагаться в корне проекта.
* **autoload** — Специфичная информация для автолоада, здесь обязательно нужно указать `registration.php` и путь до модуля, как в примере выше.

[Component types](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/prepare/dev-modtypes.html)
[The composer.json file](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/composer-integration.html)

2. Создать `etc/module.xml`

Файл содержит основную информацию о модуле:
```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="Exam_Test" setup_version="1.0.0">
        <sequence>
            <module name="Magento_Catalog"/>
        </sequence>
    </module>
</config>
```

* name — имя модуля
* setup_version — версия, начиная с 2.2 можно не указывать если используется декларативная схема (см. 4.2)
* sequence — здесь указываються слабые (soft) зависимости модуля

3. Создать `registration.php`

Технический файл необходимый для регистрации модуля в системе:
```php
<?php

\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Exam_Test',
    __DIR__
);
```

4. Запусть:
```bash
php bin/magento setup:upgrade
```

### registration.php

Для каждого типа копонента используеться различные способы регистрации см. [Register your component](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/component-registration.html)

* модуль
```php
\Magento\Framework\Component\ComponentRegistrar::register(ComponentRegistrar::MODULE, '<VendorName_ModuleName>', __DIR__);
```
>Avoid using “Ui” for your custom module name because the %Vendor%_Ui notation, required when specifying paths, might cause issues.
* тема
```php
\Magento\Framework\Component\ComponentRegistrar::register(ComponentRegistrar::THEME, '<area>/<vendor>/<theme name>', __DIR__);
```
* языковой пакет
```php
\Magento\Framework\Component\ComponentRegistrar::register(ComponentRegistrar::LANGUAGE, '<VendorName>_<packageName>', __DIR__);
```
* библиотке
```php
\Magento\Framework\Component\ComponentRegistrar::register(ComponentRegistrar::LIBRARY, '<vendor>/<library_name>', __DIR__);
```

#### Флоу загрузки

1. pub/index.php вызывает 

```php
require __DIR__ . '/../app/bootstrap.php';
```
2. app/bootstrap.php вызывает 

```php
require_once __DIR__ . '/autoload.php';
```
3. autoload.php 

```php
$vendorDir = require VENDOR_PATH;
$vendorAutoload = BP . "/{$vendorDir}/autoload.php";

/* 'composer install' validation */
if (file_exists($vendorAutoload)) {
    $composerAutoloader = include $vendorAutoload;
} else if (file_exists("{$vendorDir}/autoload.php")) {
	$vendorAutoload = "{$vendorDir}/autoload.php";
	$composerAutoloader = include $vendorAutoload;
} else {
    throw new \Exception(
        'Vendor autoload is not found. Please run \'composer install\' under application root directory.'
    );
}
```
4. vendor/autoload.php + сгенерированные композеровским автолоадом файлы

5. app/etc/NonComposerComponentRegistration.php вызываеться `main()`:

```php
function main()
{
    $globPatterns = require __DIR__ . '/registration_globlist.php';
    $baseDir = dirname(dirname(__DIR__)) . '/';

    foreach ($globPatterns as $globPattern) {
        // Sorting is disabled intentionally for performance improvement
        $files = glob($baseDir . $globPattern, GLOB_NOSORT);
        if ($files === false) {
            throw new RuntimeException("glob(): error with '$baseDir$globPattern'");
        }
        array_map(function ($file) { require_once $file; }, $files);
    }
}
```

## Ответы

### Describe module limitations

Ограничения модуля следуемые из архитектуры M2:

* Модуль должен реализовывать одну фичу
* Зависимости модулей должны быть объявлены явно, так-же это касается зависимостей от остальных компонентов
* Удаление или отключене одно модуля не должно приводить к отключению другого, если это происхдоит зависимость должна быть явно указана.

### How do different modules interact with each other?

M2 совместима с [PSR-4](https://www.php-fig.org/psr/psr-4/). Главные принципы модульного взаимодействия в M2 это:

#### Внедрение зависимостей(DI)

[О паттерне](https://designpatternsphp.readthedocs.io/ru/latest/Structural/DependencyInjection/README.html)
[Dependency injection](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/depend-inj.html)

Используется для замены функциональности предоставляемой классом `Mage` из M1. 

Внедрение зависимостей предпологает, что в объект `A` явно объявляет свои зависимости внешнему объекту `B`, который эти зависимоси предоставляет. Зависимости объявленные в `A`, как правило, должны быть интерфейсами классов, а зависимости предосталяемые `B` конкретными реаизациями. 

Таким обзраом это уменьшает связаность кода, т.к. `A` больше не заботится самостоятельно о своих зависимотях, а `B` предоставляет необходимые на основе конфига или желаемого поведения.

#### Сервисные контркты

[Service contract design patterns](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/service-contracts/design-patterns.html)

Описывает какие интефрфейсы необходимо реализовать для взаимодействия модуля. 
Включает в себя:

* Интерфейсы данных — для предоставления данных

Интерфейсы лежат в _Api/Data_, например [`Magento\Customer\Api\Data\CustomerInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Api/Data/CustomerInterface.php)

* Севисные интерфейсы — для реализации бизнес-логики

Могут быть:

* Интерфейсы репозиториев (Repository interfaces) — предоставляют доступ к объектам данных, позволяют загружать, сохранять, удалять их.
Пример: [`Magento\Customer\Api\CustomerRepositoryInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Api/CustomerRepositoryInterface.php)
* Интрефейсы управления (Management interfaces) — предоставляют функции управления, напрямую несвязанные с хранением, например изменения пароля или активация аккаунта
Пример: [`Magento\Customer\Api\AccountManagementInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Api/AccountManagementInterface.php)
* Интерфейсы метадаты (Metadata interfaces) — предоставляют различную методату, например атрибуты
Пример: [`Magento\Customer\Api\MetadataInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Api/MetadataInterface.php)

#### Зависимости модулей

[Module dependencies](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_depend.html)

Основной принцип архитектуры M2 минимизация зависимостей. 

Различаеться два типа зависимостей _сильная_ и _слабая_.

###### Сильная

Один модуль не может работать без другого, такие модули:

* Содержат код использующий логику из другого модуля: константы классов, статичные методы, публичные поля классов, интерфейсы и трейты;
* Содержат строки включающие имена классов, интерфейсов, методов, трейтов другого модуля;
* Используют объекты из другого модуля;
* Используют или изменют таблицы БД используемые в другом модуле.

Для таких модулей стоит явно прописывать зависимость в `composer.json` файле модуле в секции `require`:

```json
 ...
  "require": {
    "magento/module-catalog": "103.0.*",
    "magento/module-email": "101.0.*",
    "magento/module-media-storage": "100.3.*",
    "magento/module-store": "101.0.*",
    "magento/module-theme": "101.0.*",
    "magento/module-ui": "101.1.*",
    "magento/module-variable": "100.3.*",
    "magento/module-widget": "101.1.*",
    "magento/module-authorization": "100.3.*"
   },
   ...
```

###### Слабая 

Модуль в принципе может работать без другого модуля, в таких модулях:

* Непосредствено проверяет доступность другого модуля;
* Расширяет конфигурации другого модуля;
* Расширяет лаяут другого модуля.

Такие модули нужно объявлять в файле `etc/module.xml` в ноде `sequence`:

```xml
  <module name="Magento_Cms">
     <sequence>
        <module name="Magento_Store"/>
        <module name="Magento_Theme"/>
        <module name="Magento_Variable"/>
     </sequence>
  </module>
```

и в suggest файла composer.json.

##### Допустимые зависимости

* Другие модули M2
* PHP экстеншены
* Библиотеки, предоставляемые маджентой или сторонние

Зависимости между классами должны быть в одном модуле, между собой модули должны взаимодействовать через API интерфейсы (сервис контракты).

#### Взаимодействие модулей

[Module relationships](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_relationships.html)

Типы отношений:

* использование (uses) — модуль `A` использует модуль `B` если он вызывает `B`
* реакця (reacts to) — модуль `A` реагирует `B` если его поведение вызвано действиями в `B` при этом `B` не знает об `A`
* изменение (customizes) — `A` изменяет `B` если он модифицирует поведение `B`
* реализация (implements) — `A` реализует `B` если он реализует какое-либо поведение определённое в `B`
* замена (replaces) — `A` заменяет `B` если предоставляет свою версию API предоставленную и реализованную в `B`

#### Модули и области (areas)

area — это логический компонент предназначеный для оптимизации процеса запроса. Area в M2 подобны тем, что были в M1.

M2 имеет такие области([`Magento\Framework\App\Area`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Area.php)):

* **Admin** (`adminhtml`) — всё связанное с админкой
* **Storefron** (`frontend`) — пользовательская часть
* **Basic** (`base`) — используеться для fallback если что-то не было найдено в `adminhtml` или `frontend`
* **Cron** (`crontab`) — для `cron.php` [`Magento\Framework\App\Cron`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Cron.php) всегда загружаеться в этой области 
Так же имеються area для API
* **Web API REST** (`webapi_rest`)
* **Web API SOAP** (`webapi_soap`)
* **Graphql** (`graphql`)

Модуль определяет какие ресурсы видны и доступны в области, а также поведение. Модули могут использовать несколько областей.

Для получения списка областей используется класс [`Magento\Framework\App\AreaList`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/AreaList.php)

### Работа с областью

Класс [`Magento\Framework\App\State`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/State.php), методы:
* [`getMode()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/State.php#L108) — текщуий режим мадженты
* [`getAreaCode()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/State.php#L150) — текуная область, если не задана то выбросит исключение
* [`setAreaCode($area)`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/State.php#L131) — задать область, если уже задана, то выбросит исключене
* [`emulateAreaCode()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/State.php#L179) — эмулировать область
* [`isAreaCodeEmulated()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/State.php#L165) — эмулируется ли сейчас область

### What side effects can come from this interaction?

* Ошибки при отсутвии/отключении каких-либо модулей
* Ошибки при попытке загрузить отсутвующие классы, объекты

## Подробнее

[Module overview](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_intro.html)
