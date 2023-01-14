# CMS & Deploys

## CMS

Здесь всё аналогично M1, имеются блоки, страницы, виджеты.

### Директивы Magento

* `{{config path="general/store_information/name"}}` — значение конфига
* `{{media url="wysiwyg/sale/sale-20-off.png"}}` — ссылка на ресурс из медии
* `{{view url='Magento_Customer/images/icn_checkout.png'}}` — ссылки на ресурс из темы
* `{{store url="admin/auth/resetpassword/"}}` — получение урла из path
* `{{trans "string to translate"}}` — передов строки
* `{{block class='Magento\Framework\View\Element\Template' template='Magento_Sales::path/to/template.phtml'}}` — вывод блока с темлейтом
* `{{layout handle="sales_email_order_creditmemo_items"}}` — вывод вывода хенда(обычно используется в письмах)

## Deploys

### Режимы работы

В M2 существует 3 режима работы: `default`, `development` и `production`. С первым типом маджента поставляется изначально, и обычно этот тип не используется, а сразу меняется на два других.
* `develop` — используется на локалке разработчика
* `production` — на серверах

Различия режимов проявляется в том как маджента подготавливает и отдаёт файлы:
1. Браузер запрашивает статичный файл — стиль, картинку и т.п. из деректории
`pub/static/frontend/<Theme_Vendor>/<Theme_Name>/<locale>`.
2. Если файла нет, то поведение мадженты зависит от типа файла и режима работы:

||Developer|Production|
|-|-|-|-|
|.css и .less|Файлы .less содержащие `@magento_import` обрабатываются бекндом. На отальные `.less` создаются симлинки. `.less` компилируется фронтовым сборщиком в `.css`|`.less` компилируются в `.css`, итоговые файлы копируются. При изменении оригинальных файлов необходма пересборка|
|другие|создаются симлинки на оригинальные файлы. При изменении оригинальных файлов, файлы отдаваемые в браузер всегда актуальны|Оригинальные файлы копируются. При изменении оригиналов необходима пересборка|

### Команды сборки

* `php bin/magento setup:static-content:deploy` — применяется в production-режиме, производит компиляцию `.less` в `.css` и копирование всех файлов в `pub/static/`. Имеет множество параметров [Подробнее](https://devdocs.magento.com/guides/v2.2/config-guide/cli/config-cli-subcommands-static-view.html)
* `php bin/magento dev:source-theme:deploy` — маджента расширяет синтаксис less директивой `@magento_import` позволяющей использовать стандартный fallback-мехнанизм для `.less` файлов. Эта часть функционала выполняется через бекенд, и кладёт результат в _pub/static_. На файлы не содержащие `@magento_import` делаются симлинки. [Подробнее](https://devdocs.magento.com/guides/v2.2/config-guide/cli/config-cli-subcommands-less-sass.html)
* `grunt exec` — выполняет `php bin/magento dev:source-theme:deploy` фронтовым сборщиком
* `grunt less` — комрилирует less в css. При добавлении/удалении less нужно выполнять `grunt exec`.
* `grunt watch` — выполнение `grunt less` в live-режиме.

## Кастомный сборщик

Дефолтный сборщик в мадженте работает очень медлено и имеет ряд недостаикрв, поэтому вместо него используется кастомный сборщик.
