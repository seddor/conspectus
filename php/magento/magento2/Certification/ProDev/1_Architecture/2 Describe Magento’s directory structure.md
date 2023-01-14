# Describe Magento’s directory structure

Различные типы компонентов в M2 имеют разную файловую структуру и требуют определённых файлов.

## Корневая директория

Корневая директория компонента совпадает с именем компонента и содержит все его поддериктории и файлы.

Обычно в M2 копоненты могут располагаться в двух директориях:

* _app_ — рекомендуемая директория для разрабатываемых компонентов:

  * _app/code_ — для модулей

  * _app/design/frontend_ — для фронтовых тем

  * _app/design/adminhtml_ — для админских тем

  * _app/i18n_ — для языковых пакетов
* _vendor_ — для компонентов поставленных через composer

## Обязательные файлы

Все компоненты должны обязательно содержаь эти файлы:

* `registratino.php` — файл регистрирует компонент в системе. [Register your component](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/component-registration.html)
* `composer.josn` — **(обязателен только для composer-компонетов)** содержит зависмости (сильные) модуля. [The composer.json file](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/composer-integration.html)

## Структура модуля

* `Block` — PHP view-классы (MVC)
* `Controller` — PHP классы-controller (MVC)
* `Controller/Adminhtml` — PHP контролеры для админки
* `etc` — файлы конфигурации, в частности обязательнй `module.xml`, используется xml
* `Model` — PHP model-классы (MVC)
* `Model/ResourceModel` — PHP ресурсные модели
* `Setup` — PHP классы для измнения структуры БД и данных вызваються при установке обновлении
* `Api` — PHP интерфесы используемые для API, а так-же сервис контрактов
[Service contract anatomy](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/service_layer.html#service-contract-anatomy)
* `Console` — PHP файлы для CLI-комманд 
[Command line configuration](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli.html#config-new-cli-intro)
[Add CLI commands](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/cli-cmds/cli-add.html)
* `Cron` — PHP классы для cron job
* `CustomerData` — PHP классы реализуемые [`Magento\Customer\CustomerData\SectionSourceInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/CustomerData/SectionSourceInterface.php) которе передают датус бекенда на фронтенд. 
[Private content](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/cache/page-caching/private-content.html)
* `Helper` — PHP классы хелперов
* `i18n` — csv файлы для локализации
[Translation dictionaries](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/translations/xlate.html#m2devgde-xlate-dictionaries)
* `Observer` — PHP классы для обработчиков событий
* `Plugin` — PHP плагины, новая фича M2. Плагин позволяет встраивать код перед/после/вокруг публичного метода класса. По сути заменяет функционал реврайтов из M1.
[Plugins (Interceptors)](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/plugins.html)
* `Ui` — PHP UI классы [Overview of UI components](https://devdocs.magento.com/guides/v2.4/ui_comp_guide/bk-ui_comps.html)
* `Test` — юинит тесты
* `view` — view-файлы, в том числе статику, темплейты, email темплейты и лаяуты:
  * _view/`<area>`/email_
  * _view/`<area>`/layout_
  * _view/`<area>`/templates_
  * _view/`<area>`/ui_component_
  * _view/`<area>`/ui_component/templates_
  * _view/`<area>`/web_
  * _view/`<area>`/web/template_
  * _view/`<area>`/requirejs-config.js_

### Обязательные файлы

* `registration.php`
* `composer.json` — только для composer-компонетов
* `etc/module.xml` — содержит базовую информацию о комоненте, его зависимостях (слабых), версию комонента, она используеться для определения используемых инсталеров (как в M1, и если не используется декларативная схема), инсталлеры запускаються по команде `bin/magento setup:upgrade`

## Структура темы

[Magento theme structure](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/themes/theme-structure.html)
[Темы (раздел из лекций "Разрабатывай фронт для М2 правильно")](https://docs.google.com/document/d/1coMy9ot22x73j3nYAviJIgcJwkgn0D551lbo6VS6cyQ/edit#heading=h.a1cwxdu8gsws)

* `etc` — файлы конфигурации.
* `i18n` — csv файлы локализации
* `media` — содержит картинку с превью темы
* `web` — содержит статичные файлы:
  * `css/source` — less файлы
  * `css/source/lib` — заоверайденые классы маджентовской [UI библиотеки](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/css-topics/theme-ui-lib.html)
  * `fonts` — шрифты
  * `images` — изображения
  * `js` — JS файлы

### Обязательные файлы

* `registration.php`
* `composer.json` — только для composer-компонетов
* `theme.xml` — содержит название темы, родительскую тему (если есть) и картинку-превью для админки (скриншот страницы сайта, обычно media/preview.jpg)
```xml
<theme xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Config/etc/theme.xsd">
    <title>Magento Luma</title>
    <parent>Magento/blank</parent>
    <media>
        <preview_image>media/preview.jpg</preview_image>
    </media>
</theme>
```
* `etc/view.xml` — обязателен для темы, но не обязателен если есть в родительских - задает размеры продуктовых картинок и некоторые другие конфигурации.

## Структра языкового пакета

[Translation dictionaries and language packages](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-i18n.html)

пример:
```
├── de_DE
│   ├── composer.json
│   ├── language.xml
│   ├── LICENSE_AFL.txt
│   ├── LICENSE.txt
│   └── registration.php
├── en_US
│   ├── composer.json
│   ├── language.xml
│   ├── LICENSE_AFL.txt
│   ├── LICENSE.txt
│   └── registration.php
├── pt_BR
│   ├── composer.json
│   ├── language.xml
│   ├── LICENSE_AFL.txt
│   ├── LICENSE.txt
│   └── registration.php
```

### Обязательные файлы

* `registration.php`
* `composer.json` — только для composer-компонетов
* `language.xml` — содержит:
```xml
<language xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/Language/package.xsd">
    <code>en_GB</code>
    <vendor>magento</vendor>
    <package>en_gb</package>
    <sort_order>100</sort_order>
    <use vendor="oxford-university" package="en_us"/>
</language>
```
 * `code` — (обязательно) локаль
 * `vendor` — (обязательно)
 * `package` — (обязательно) название пакета 
 * `sort_order` — приоритет загрузки пакета среди друних языкрв стора
 * `use` — родительский пакет, может быть несколько:
```xml
<use vendor="parent-package-one" package="language_package_one"/>
<use vendor="parent-package-two" package="language_package_two"/>
```

## Подробнее

[About component file structure](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/prepare/prepare_file-str.html)
[Create your component file structure](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/module-file-structure.html)
