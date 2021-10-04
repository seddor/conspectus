# Front controller

Front controller ([`Magento\Framework\App\FrontController`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/FrontController.php)) — используется для эффективого управления запросами и организации прямого рабочего флоу для всех страниц.

## Предназначение

Front controller отвечает за:

* Получения всех роутеров (внедряются через конструктор используя DI)
* Нахождение подходящего контроллера/роутера
* Получение сгенерированного вывода для объекта ответа

M2 предоставляет набор роутеров и обычно их достаточно. 

В M2 реврайт URL реализуется через роутер.

## Высокоуровневая схема процеса выполнения

1. index.php
2. `Bootstrap::run()` 
3. `App::launch()`
4. `FrontController::dispatch()`
5. `Router::match()` (цикл по роутерам)
6. `Controller::execute()`
7. `View::loadLayout()`
8. `View::renderLayout()`
9. `Response::sendResponse()`

# Роутинг

## Список роутеров

Список роутеров получется в конструкторе front controller через DI, путем передачи параметра `$routerList` с типом [`Magento\Framework\App\RouterListInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/RouterListInterface.php).

Для роутинга в M2 используются классы реализующие [`Magento\Framework\App\RouterInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/RouterInterface.php)

В M2 содержится 5 основых роутеров (в порядке выполненя поиска):

* [`standard`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Router/Base.php) — стандартный роутер для magento-style URL
* [`default`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Router/DefaultRouter.php) — no-route
* [`urlrewrite`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/UrlRewrite/Controller/Router.php) — матчит по реврайтам из БД
* [`robots`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Robots/Controller/Router.php) — матчт по `robots.txt`.

В каждом роутере выполянется метод `match()`, пока какой-нибудь роутер не вернёт ответ или не будет достигнут лимит в 100 итераций (тогда будет выброшено исключение).

## Регистрация нового роутера

Для регистрации нового роутера нужно добавить его в качестве параметра в класс [`Magento\Framework\App\RouterList`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/RouterList.php), это можно сделать через `di.xml`.

Например, так регистрируется роутер для работы с CMS-страницами:
```xml
    <type name="Magento\Framework\App\RouterList">
        <arguments>
            <argument name="routerList" xsi:type="array">
                <item name="cms" xsi:type="array">
                    <item name="class" xsi:type="string">Magento\Cms\Controller\Router</item>
                    <item name="disable" xsi:type="boolean">false</item>
                    <item name="sortOrder" xsi:type="string">60</item>
                </item>
            </argument>
        </arguments>
    </type>
```

# Работа с URL

## Стандартные URL

Стандартные URL обрабатываются классом [`Magento\Framework\App\Router\Base`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Router/Base.php).

Стандартные урлы имеют такую структуру:

>/catalog/product/view/id/1

Здесь:

* Front name: catalog
* Controller name: product
* Action name: view
* Параметры: id=1

### Нестандартные роутеры

Многие роутеры работающие с нестандартными урлами обрабатывают URL и приводят его к стандартному виду. Например CMS-роутер в итоге делает:
```php
$request->setModuleName('cms')
    ->setControllerName('page')
    ->setActionName('view')
    ->setParam('page_id', $pageId);
```
