# Utilize JavaScript in Magento

>Describe different types and uses of JavaScript modules. Which JavaScript modules are suited for which tasks?

>Describe UI components. In which situation would you use UiComponent versus a regular JavaScript module?

>Describe the use of `requirejs-config.js`, `x-magento-init`, and `data-mage-init`.

## Расположение и fallback

Для файлов в теме:

* _`<theme_dir>`/web/i18n/`<locale>`/js/…_
* _`<theme_dir>`/web/js/…_
* _`<parent_theme_dir>`/web/i18n/`<locale>`/js/…_
* _`<parent_theme_dir>`/web/js/…_
* _lib/web/…_

Для файлов в модуле:

* _`<theme_dir>`/web/i18n/`<locale>`/`<Namespace_Module>`/js/…_
* _`<theme_dir>`/`<Namespace_Module>`/web/js/…_
* _`<parent_theme_dir>`/web/i18n/`<locale>`/`<Namespace_Module>`/_
* _js/…_
* _`<parent_theme_dir>`/`<Namespace_Module>`/web/js…_
* _`<module_dir>`/view/frontend/web/js…_
* _`<module_dir>`/view/base/web/js…_

Обычно для хранения js-файлов используется подкаталог _web/js_, но в ядре бывают и исключения.

## RequireJS

RequireJS работает по схеме AMD (асинхронное определение модуля), при таком подходе JS-модули и заивисимости загружаются асинхронно. 
Для работы с requireJs в M2 нужно создать JS-модуль, он создаётся как js-файл с подобным содержимым:
```js
define([...], function(...) {
    return ...;
});
```

Функция `define` регистрирует наш код как модуль, который необходимо куда-нибудь подключить, т.к. сам выполнятся он не будет. Первым параметром `define` принимает массив зависимостей, вторым — нашу функцию. Наша функция на вход принимает список загруженных зависимостей, а вернуть она должна непосредственно объект-модуль.
Помимо `define`, мы можем использовать функцию `require`. Получится js-файл подобного содержания:

```js
require([...], function(...) {
    return ...;
});
```
`require` принимает такие же параметры как и `define`, однако отличие в том, что `define` служит для определения модуля, а `require` является точкой входа для выполнения скрипта.
Обычный кейз - несколько js-файлов определяют модуля через `define`, а `require`, помещенный прямо в html коде, подключает и использует их.

## Подключение JS

* Через лаяут, директоивой `<script>`. Происохдит внутри директивы `<head>`, используется редко, обычно если скрипт не может быть подключен как RequireJS-зависимость
```xml
<script src="requirejs/require.js"/>
```
* Через ноду `deps` в `requirejs-config.js`. Скрипт подключается на все станицы, таким путём можно подключать глобальные библиотеки, используемые на всём сайте, например jQuery:
```js
deps: [
    'jquery/jquery.mobile.custom',
    'mage/common',
    'mage/dataPost',
    'mage/bootstrap'
]
```
* через аргумент `data-mage-init` в html-коде — расширение библиотеки knockout js, аттрибут принимает json-строку в виде объекта, ключи которого модули, а значение — конфигурация. Если модуль объявлен как функция, то она вызывается после инициализации js-ядра маженты. Первым параметром передается конфигурация из json-строки, вторым — элемент. 
Также, очень часто, вместо обычного js-модуля инициализируется jQuery-виджет. Пример:
```html
<form data-mage-init='{"validation": {"errorClass": "mage-error"}}'>
```
* через аргумент `data-bind` с инструкцией `mageInit` в html-коде — является стандартным инструментом биндинга в библиотеке knockout js. А использование директивы `mageInit` позволяет подключать модули по аналогии с `data-mage-init`. Пример:
```html
<span data-bind="mageInit: {'dropdown':{'activeClass': '_active'}}">
```
Основное отличие двух вышеописанных методов в том, что по умолчанию data-mage-init инициализируется один раз, при загрузке страницы, а data-bind каждый раз при обработке knockout’ом. Т.е. в phtml шаблонах используем data-mage-init, а в knockout-шаблонах data-bind с mageInit. Если же необходимо вставлять новый html код, например, через jQuery-шаблонизатор, можно использовать data-mage-init, а после вставки кода вызывать метод apply у модуля ‘mage/apply/main’. После инициализации атрибуты data-mage-init удаляются, и повторно инициализироваться не будут.
* через ноду `<script type="text/x-magento-init">` в html-коде. Скрипт должен быть представлен в виде json’a, структуру которого легче объяснить на примере:
```html
<script type="text/x-magento-init">
    "[data-container=new-password]": {
        "passwordStrengthIndicator": {
            "formSelector": "form.form-edit-account"
        }
    }
</script>
```
В представленном выше примере, всем элементам на странице с селектором [data-container=new-password] применяться указанные js-модули с указанной конфигурацией. Как можно заметить, данный способ похож на использование data-mage-init, однако здесь мы можем назначать js-модуль целому ряду элементов.
У данного способа есть одна особенность: если в качестве селектора указать звездочку “*”, js-модуль применяется, однако без привязки к элементу. Стоит помнить, что несмотря на то, что селектор звездочка выбирает все элементы на страницы, в данном случае он обозначает выполнение js-модуля без привязки к какому-либо элементу. Пример:
```html
<script type="text/x-magento-init">
    {
        "*": {
            "Magento_Bundle/js/bundle-type-handler": {}
        }
    }
</script>
```
Как и в случае с data-mage-init, указанный модуль должен возвращать функцию, принимающую на вход конфигурацию и элемент.
* через использование `require([...])` в html-коде, базируется на использовании метода require библиотеки RequireJS. Как было сказано ранее, require исполняется сразу, и поэтому мы можем использовать её прямо в html-коде страницы. Пример:
```html
<script>
require(['jquery'], function($){
    var priceType = $('#price_type');
    var priceWarning = $('#dynamic-price-warning');
        if (priceType && priceType.val() == 0 && priceWarning) {
        priceWarning.show();
    }
});
</script>
```
