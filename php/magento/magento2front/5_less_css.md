# LESS & CSS

В M2 используется препроцессов CSS — LESS. Less/css может находится в контексте модуля и темы. Файлы относятся к статике, fallback для них строится следующим образом:
Для файлов в контексте темы:
1. `<theme_dir>/web/i18n/<locale>/css/`
2. `<theme_dir>/web/css/`
3. `<parent_theme_dir>/web/i18n/<locale>/css/`
4. `<parent_theme_dir>/web/css/`
5. `lib/web/css/`
Для файлов в контексте модуля:
1. `<theme_dir>/web/i18n/<locale>/<Namespace_Module>/css/`
2. `<theme_dir>/<Namespace_Module>/web/css/`
3. `<parent_theme_dir>/web/i18n/<locale>/<Namespace_Module>/css/`
4. `<parent_theme_dir>/<Namespace_Module>/web/css/`
5. `<module_dir>/view/frontend/web/css/`
6. `<module_dir>/view/base/web/css/`

В дефолтной мадженте каталоги контекста темы могут содержать некторые файлы, которые являются точками входа, т.е. такой less-файл подключает в себя другие, и в конечном итоге компилируется в обособленный css-файл.

* `print.less` — точки входа для печатной версии сайта.
* `_styles.less` — основной файл включающий в себя все less-файлы темы.
* `styles-m.less` — точка входа для мобильных стилей, включает `_styles.less` по умолючанию, подключается на всех страницах с аттрибутом `media="all"`
* `styles-l.less` — точка входа для дестопных стилей, включает `_styles.less`, по умолчани, подключается для страниц с атрибутом `media="screen and (min-width: 768px)"`.
* `source/_theme.less` — хранит новые и перезаписывает дефолтные less-переменные
* `source/_extend.less` — файл расширяющий родительскую тему, используется когда нужны небольшие измения в родительских стилях
* `source/_extends.less` — содержит миксины и подключается с аттрибутом (reference), т.е. используется по требованию, как библиотека.
* `email.less, email-fonts.less, email-inline.less` — точки входа для стилей писем.

Каталоги контекста модулей могут содержать файл `source/_module.less` — корневой less файл модуля, содержащий собственные стили, или подключащий файлы, лежащие рядом.
Все директории `web/css` могут содежрать каталог `source` для основных less файлов, непосредствено для стлизации сайта, которые следуюет так-же называть с подчёркивания.

## Использование `@magento_import`

Данная директива обрабатывается бекендом и может применятся только при импользовании дефолтных маджентовский сборщиков.

## Подключение стилей

Для полючения css-файла на страницу, необходимо в `<head>` page-лаяута добавить инструкцию `<css>`:
* подключение в контексте текущей темы:
```XML
<css src="css/styles-m.css"/>
```
Использование media условий:
```XML
 <css src="css/styles-l.css" media="screen and (min-width: 768px)"/> или <css src="css/print.css" media="print" />
```
* подключение файла в контексте модуля:
```XML
<css src="Magento_Swatches::css/swatches.css"/>
```
* удаление подключенного файла:
```XML
<remove src="css/print.css"/>
```
Пр инеобходимости подключения файла на все страницы стоит использовать хендл: `default_head_blocks`.

## Magento UI libary

Маджента имеет собственную ui-библиотеку стилей.
[Полный список компонентов](https://devdocs.magento.com/guides/v2.2/frontend-dev-guide/css-topics/theme-ui-lib.html#library_elements)
