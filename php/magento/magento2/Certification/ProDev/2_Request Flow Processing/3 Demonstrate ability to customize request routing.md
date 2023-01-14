# Demonstrate ability to customize request routing

>Describe request routing and flow in Magento. When is it necessary to create a new router or to customize existing routers? How do you handle custom 404 pages?

[Routing](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/routing.html)

Для роутинга в M2 используются классы реализующие [`Magento\Framework\App\RouterInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/RouterInterface.php)

Для `frontend` это:
* [`robots`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Robots/Controller/Router.php) — матчит по `robots.txt`.
* [`urlrewrite`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/UrlRewrite/Controller/Router.php) — матчит по реврайтам из БД
* [`standard`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Router/Base.php) — стандартный роутер, матчит по маджентовской 3х уровневой структуре урла
* [`cms`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Cms/Controller/Router.php) — матчит по CMS страницам
* [`default`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Router/DefaultRouter.php) — используется если других не было найдено

Для `adminhtml` это:
* [`admin`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/App/Router.php) — наследуется от `standard`
* [`default`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Router/DefaultRouter.php)

## standard

Сохже с M1.

```
<store-url>/<store-code>/<front-name>/<controller-name>/<action-name>
```

* `<store-url>` — базовый урл
* `<store-code>` — код стора (если есть)
* `<front-name>` — frontName из `routers.xml`
* `<controller-name>` — имя контроллера
* `<action-name>` — класс с экшеном


## default

Вызывается если не было заматчено ни одного роута, по дефолту возвращает [`Magento\Framework\App\Route\NoRouteHandlerInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Router/NoRouteHandlerInterface.php)

## Кастомне роуты

1. Реализовать [`Magento\Framework\App\RouterInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/RouterInterface.php)
2. Прописать новый роут в `di.xml`:
```xml
<type name="Magento\Framework\App\RouterList">
    <arguments>
        <argument name="routerList" xsi:type="array">
            <item name="%name%" xsi:type="array">
                <item name="class" xsi:type="string">%classpath%</item>
                <item name="disable" xsi:type="boolean">false</item>
                <item name="sortOrder" xsi:type="string">%sortorder%</item>
            </item>
        </argument>
    </arguments>
</type>
```
* `%name%` — уникальное имя
* `%classpath%` — класс роутера
* `%sortorder%` — сорт ордер в списке роутеров

## routers.xml

все frontName пишутся в `routers.xml`

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="%routerId%">
        <route id="%routeId%" frontName="%frontName%">
            <module name="%moduleName%"/>
        </route>
    </router>
</config>
```
* `%routerId` — идентификатор роутера
* `%routeId%` — идентификатор роута
* `%frontName%` — front_name секция URL
* `%moduleName%` — имя используемого модуля

### Before and after

Можно так-же задавать параметры `before` или `after` для переопределения или расширения существующего роута:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="standard">
        <route id="customer">
            <module name="OrangeCompany_RoutingExample" before="Magento_Customer" />
        </route>
    </router>
</config>
```

## How do you handle custom 404 pages?

* 404 вызывается если выбрасывается экзепшен [`Magento\Framework\Exception\NotFoundException`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Exception/NotFoundException.php)
* метод для получения экшена 404: [`Magento\Framework\App\Router\Base::getNotFoundAction()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Router/Base.php#L246) он пытаеться вызвать `noroute` у последнего использованного модуля
* если у модуля нет `noroute`, то:
  * используется экшен, который устанавливается в админке ([`Magento\Framework\App\Router\NoRouteHandler::process()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Router/NoRouteHandler.php#L32))
  * используются кастомные хендлы зарегистрированные в [`Magento\Framework\App\Router\NoRouteHandlerList`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Router/NoRouteHandlerList.php)
