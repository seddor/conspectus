# Работа с контроллерами

## Процесс поиска контролерами

Процесс поиска происходит по такому алгоритму:

1. Определить спсисок назначенных модулей
2. Проверить каждый модуль, построить потенциальный путь до контроллера
3. Проверить наличие контроллера
    * если есть, то вернуть его и остановить процесс поиска
    * если нет, то вернутся к 2.

### Base router

[`Base`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Router/Base.php)

* основной роутер в M2, он находит большинство экшенов
* Содержит метод `matchAction()` где обрабатывается почти каждый экшен (кроме реврайтов, они обрабатываются другим роутером)
* каждый урл, обрабатываемый базовым роутером, обрабатывается по Magento-style структуре: _frontName/actionName/action + параметры_
    * frontName — задаётся в `routers.xml` определяет какой модуль соответствует frontName. 
    * actionName ­— поддиректория в _`<module-dir>/Controller`_.
    * action — класс в поддиректориях контроллеров.
    * все остальные секции воспринимаются как параметры

## Создание контроллера

1. Создать `routers.xml`:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="standard">
        <route id="catalog" frontName="catalog">
            <module name="Magento_Catalog" />
        </route>
    </router>
</config>
```
2. Создать класс-контролера (должен наследоватся от [`Magento\Framework\App\Action\Action`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Action/Action.php) для фронта, и от [`Magento\Backend\App\Action`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Backend/App/Action.php) для бекенда), реализовать в нём метод `execute()`.

### Кастомизация

Для кастомизации можно использовать preference (в di.xml) или плагины.

