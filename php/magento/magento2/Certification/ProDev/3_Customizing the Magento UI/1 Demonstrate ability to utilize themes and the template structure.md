# Demonstrate ability to utilize themes and the template structure

>Demonstrate the ability to customize the Magento UI using themes. When would you create a new theme? How do you define theme hierarchy for your project?

>Demonstrate the ability to customize/debug templates using the template fallback process. How do you identify which exact theme file is used in different situations? How can you override native files?

[Utilizing Themes and the Template Structure in Magento 2](https://belvg.com/blog/utilizing-themes-and-the-template-structure-in-magento-2.html)

## Создание темы

[Magento 2: Creating a New Theme ](https://belvg.com/blog/magento-2-creating-a-new-theme.html)
[Как создать тему для Magento 2 с нуля](https://habr.com/ru/post/311350/)

Темы в M2 представляют из себя компонет, и могут храниться в:

* для composer-тем: в _vaendor/`<vendor>`/theeme-`<area>`-`<name>`_
* в _app/design/frontend/`<vendor>`/`<name>`_

Обязательные составляющие темы:

* `registration.php`:
```php
\Magento\Framework\Component\ComponentRegistrar::register(ComponentRegistrar::THEME, 'frontend/Magento/luma', __DIR__);
```
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

## Иерархия тем

Родительская тема задаётся в `theme.xml`, в `<parent>`:
```xml
<parent>Magento/blank</parent>
```
Обычно в качестве базовой родительской темы используется [`Magento/blank`](https://github.com/magento/magento2/tree/2.4/app/design/frontend/Magento/blank). Иерархия используется для fallback.

## Кастомизация темплейтов

[Overriding Layout Files in Magento 2](https://belvg.com/blog/overriding-layout-files-in-magento-2.html)

Для кастомизации темплейтов родительской темы в дочерней по такому же пути создаётся темлейт с такимже именем, он переопределяет родительский.

## Хинты с путями темлейтов

Как и в M1, в M2 можно включить хинты с путями темплейтов и именами блоков, их можно включить в админке по пути _Stores — Configuration — Advanced — Developer — Debug_.

Или командой:

```bash
php bin/magento dev:template-hints:enable
```

После этого нужно почистить кэш, для отключаения используется команда:

```bash
php bin/magento dev:template-hints:disable
```



## Загрузка тем

* Плагин в [`magento-store/etc/di.xml`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Store/etc/di.xml#L68)
* [`Magento\Framework\App\Action\Plugin\LoadDesignPlugin::beforeExecute()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Action/Plugin/LoadDesignPlugin.php#L50) 
```php
    public function beforeExecute(ActionInterface $subject)
    {
        try {
            $this->_designLoader->load();
        } catch (LocalizedException $e) {
            if ($e->getPrevious() instanceof ValidationException) {
                //...
            }
        }
    }
```
* [`Magento\Framework\View\DesignLoader::load()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/DesignLoader.php#L51)
```php
    public function load()
    {
        $area = $this->_areaList->getArea($this->appState->getAreaCode());
        $area->load(\Magento\Framework\App\Area::PART_DESIGN);
        //...
    }
```
* [`Magento\Framework\App\Area::load()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Area.php#L138)
* [`Magento\Framework\App\Area::_loadPart()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Area.php#L202)
* [`Magento\Framework\App\Area::_initDesign()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Area.php#L259)
```php
    protected function _initDesign()
    {
        $this->_getDesign()->setArea($this->_code)->setDefaultDesignTheme();
        return $this;
    }
```
  *  [`Magento\Theme\Model\View\Design`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Theme/Model/View/Design.php)
  * design.setArea('frontend')
  * design.getConfigurationDesignTheme
  * design.setDefaultDesignTheme 
    * из конфига `design/theme/theme_id`
