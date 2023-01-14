# Структура фронтенда

В M2 применяется такая же блочная структура как в M1, т.е. имеется дерево блоков, где каждый блок содержит свои дочерние блоки. 

Рендеринг проиходит с корневого блока путём обхода в глубину, т.е. для отрисовки родительского блока отрисовываются дочерние.

# Темы

Темы применяются аналогично M1, для измения внешнего вида сайта или его частей.

Темы бывают для админки и для фронтовой части магазина. 

Для фронтовой части есть базовая тема `Blank`, аналогичная `base` из M1.

В M2 тему можно ставить как композер-компонент. В таком случае тема будет хранится в _vendor/<vendor\>/theme-frontend-<themename\>_
Также тему можно расположить локально в проекте в директории: _app/design/frontend/<vendor\>/<themename\>_. 

## Обязательные составляющие 

* theme.xml — лежит в корне, содержит описание темы:
```xml
<theme xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Config/etc/theme.xsd">
    <title>Magento Luma</title>
    <parent>Magento/blank</parent>
    <media>
        <preview_image>media/preview.jpg</preview_image>
    </media>
</theme>
```
    * title — название темы
    * parent — родительская тема
    * media -> preview_image — превью в админке
* registration.php — файл для регистрации темы в системе
```php
<?php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::THEME,
    'frontend/Magento/luma',
    __DIR__
);

```
* composer.json — описание для композера, обязателен для тем распостраняемых через композер
```json
{
    "name": "magento/theme-frontend-luma",
    "description": "N/A",
    "require": {
        "php": "~7.0.13|~7.1.0",
        "magento/theme-frontend-blank": "100.2.*",
        "magento/framework": "101.0.*"
    },
    "type": "magento2-theme",
    "version": "100.2.5",
    "license": [
        "OSL-3.0",
        "AFL-3.0"
    ],
    "autoload": {
        "files": [
            "registration.php"
        ]
    }
}
```
* etc/view.xml — обязателен для темы, но не обязателен, т.к. может наследоваться из родительской темы, в нём задаются размеры продуктовых картинок и некоторые другие конфигурации.
```xml
<?xml version="1.0"?>
<view xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Config/etc/view.xsd">
    <media>
        <images module="Magento_Catalog">
            <image id="product_small_image" type="small_image">
                <width>135</width>
                <height>135</height>
            </image>
            <image id="product_thumbnail_image" type="thumbnail">
                <width>75</width>
                <height>75</height>
            </image>
        </images>
    </media>
</view>
```

## Модули

Файлы фронтенда могут храниться не только в теме, но и в катлогах модуля:
* `<module_dir>/view/frontend`
* `<module_dir>/view/base`
* `<module_dir>/view/adminhtml`
Файлы из модуля не зависят от темы, чтобы их измениять/добавлять нужно поместь новые файлы в 
`<theme_dir>/<Namespace_Module>/`.

## Fallback

В отличии от M1 родительская тема задаётся в самой теме. 

Путь fallback примерно такой:
1. `<theme_dir>/web/i18n/<locale>/`
2. `<theme_dir>/web/`
3. `<parent_theme_dir>/web/i18n/<locale>`
4. `<parent_theme_dir>/web`
5. `lib/web`
Для файлов в контексте модуля:
1. `<theme_dir>/web/i18n/<locale>/<Namespace_Module>`
2. `<theme_dir>/<Namespace_Module>/web`
3. `<parent_theme_dir>/web/i18n/<locale>/<Namaspce_Module>`
4. `<parent_theme_dir>/<Namespace_Module>/web`
5. `<module_dir>/view/frontend/web`
6. `<module_dir>/view/base/web`
