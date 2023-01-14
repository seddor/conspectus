# Demonstrate ability to process URLs in Magento

>Describe how Magento processes a given URL. How do you identify which module and controller corresponds to a given URL? What is necessary to create a custom URL structure?

>Describe the URL rewrite process and its role in creating user-friendly URLs. How are user-friendly URLs established, and how are they customized?

>Describe how action controllers and results function. How do controllers interact with another? How are different response types generated?

## Получения URL

URL билдер реализует [`Magento\Framework\UrlInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/UrlInterface.php)

Имплементации:

* [`Magento\Framework\Url`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Url.php) — основной
* [`Magento\Backend\Model\Url`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/Model/Url.php) — для админки

Метод [`Magento\Framework\Url::getUrl($routePath = null, $routeParams = null)`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Url.php#L836):

1. Если переданный `$routePath` уже URL, то он ворвращется без изменений:
```php
if (filter_var($routePath, FILTER_VALIDATE_URL)) {
    return $routePath;
}
```
2. происходит процессинг параметров
```php
$routeParams = $this->routeParamsPreprocessor
    ->execute($this->_scopeResolver->getAreaCode(), $routePath, $routeParams);
```
routeParamsPreprocessor реализует [`Magento\Framework\Url\RouteParamsPreprocessorInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Url/RouteParamsPreprocessorInterface.php).

Имплиментация: [`Magento\Framework\Url\RouteParamsPreprocessorComposite`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Url/RouteParamsPreprocessorComposite.php)

В ней реализуется метод `execute($areaCode, $routePath, $routeParams)`:
```php
public function execute($areaCode, $routePath, $routeParams)
{
    foreach ($this->routeParamsPreprocessors as $preprocessor) {
        $routeParams = $preprocessor->execute($areaCode, $routePath, $routeParams);
    }
    return $routeParams;
}
```
routeParamsPreprocessors массив с [`Magento\Framework\Url\RouteParamsPreprocessorInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Url/RouteParamsPreprocessorInterface.php)

3. Если в `$routeParams` попадает объект, то кеш отключается и вызывается `createUrl($routePath, $routeParams)`:
```php
$isCached = true;
$isArray = is_array($routeParams);
if ($isArray) {
    array_walk_recursive(
        $routeParams,
        function ($item) use (&$isCached) {
            if (is_object($item)) {
                $isCached = false;
            }
        }
    );
}
if (!$isCached) {
    return $this->getUrlModifier()->execute(
        $this->createUrl($routePath, $routeParams)
    );
}
```
о `createUrl` ниже
4. URL берёться из кеша или вызвается [`createUrl($routePath, $routeParams)`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Url.php#L889):
```php
$cachedParams = $routeParams;
if ($isArray) {
    ksort($cachedParams);
}
$cacheKey = sha1($routePath . $this->serializer->serialize($cachedParams));
if (!isset($this->cacheUrl[$cacheKey])) {
    $this->cacheUrl[$cacheKey] = $this->getUrlModifier()->execute(
        $this->createUrl($routePath, $routeParams)
    );
}
return $this->cacheUrl[$cacheKey];
```

### createUrl

[`createUrl()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Url.php#L889)

Создаёт URL по фрагментам, процесс схож с тем, что было в M1.

Обрабатываются системные параметры, затем вызвается [`getRouteUrl($routePath = null, $routeParams = null)`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Url.php#L737), он возвращает:

* Если переадан параметр `_direct`, то `$this->getBaseUrl() . $routeParams['_direct']`.
* Иначе `$this->getBaseUrl($routeParams) . $this->_getRoutePath($routeParams)`

затем к URL добаляется строка запроса и фрагмент (если передано).

### Системные параметры

Как и в M1, в M2 можно передать системные параметры в `$routeParams`, в принципе параметры используются те же и их функционал в основном идентичен.

## Опрделения модуля и контролера

Структуры URL, такая же как была в M1, т.е.

```
route_name/controller_name/action_name/param1/value1
```

* Все доступные роуты прописываются в файлах `etc/<area>/routers.xml`
* контроллер и экшен ищется в соотвующем модуле

## Определение кастомной структуры

[Routing: Custom routers](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/routing.html#custom-routers)

Для определения необходимо:

1. Создать кастомную имплементацию [`Magento\Framework\App\RouterInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/RouterInterface.php)
2. Добавить роутер в `Magento\Framework\App\RouterList` в di.xml:
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

## Реврайты и юзер-френдли URLs

[How URL Rewrites Work in Magento 2](https://belvg.com/blog/how-url-rewrites-work-in-magento-2.html)

Роутер реврайтов: [`Magento\UrlRewrite\Controller\Router`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/UrlRewrite/Controller/Router.php)

В нём в методе `match()` происходит поиск реврайтов через [`Magento\UrlRewrite\Model\UrlFinderInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/UrlRewrite/Model/UrlFinderInterface.php), если ничего не найдено продолжается обычный процесс роутинга, иначе редирект на нужный урл.

## Кастомизация и генерация реврайтов

Для модуля `UrlRewrite`: можно использовать [`Magento\UrlRewrite\Model\UrlPersistInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/UrlRewrite/Model/UrlPersistInterface.php)

Продукт:
* ивент `catalog_product_save_before`, генерируется из имени, если url_key не был предоставлен:
  * [`Magento\CatalogUrlRewrite\Observer\ProductUrlKeyAutogeneratorObserver`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogUrlRewrite/Observer/ProductUrlKeyAutogeneratorObserver.php)
  * [`Magento\CatalogUrlRewrite\Model\ProductUrlPathGenerator`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogUrlRewrite/Model/ProductUrlPathGenerator.php#L135)
*  ивент `catalog_product_save_after`, замена при изменении url_key, категорий, видимости, сайта):
  * [`Magento\CatalogUrlRewrite\Observer\ProductProcessUrlRewriteSavingObserver`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogUrlRewrite/Observer/ProductProcessUrlRewriteSavingObserver.php)
  * [`Magento\CatalogUrlRewrite\Model\ProductUrlRewriteGenerator`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogUrlRewrite/Model/ProductUrlRewriteGenerator.php#L128)
Катеория:
* ивент `catalog_category_save_before`, генерация url_key, измение дочерних категоий:
  * [`Magento\CatalogUrlRewrite\Observer\CategoryUrlPathAutogeneratorObserver`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogUrlRewrite/Observer/CategoryUrlPathAutogeneratorObserver.php)
* ивент `catalog_category_save_after`, при изменеии url_key, якоря, продуктов:
  * [`Magento\CatalogUrlRewrite\Observer\CategoryProcessUrlRewriteSavingObserver`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogUrlRewrite/Observer/CategoryProcessUrlRewriteSavingObserver.php)

## Describe how action controllers and results function

[`App\Action\Action::dispatch()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Action/Action.php#L102)

* event `controller_action_predispatch`
* event `controller_action_predispatch_$routeName`
* event `controller_action_predispatch_$fullActionName`
* останавливается если `FLAG_NO_DISPATCH`
* `$this->execute()` - реализуют все дочерние классы контроллера
* останавливается если `FLAG_NO_DISPATCH`
* event `controller_action_postdispatch_$fullActionName`
* event `controller_action_postdispatch_$routeName`
* event `controller_action_postdispatch`
* если `$this->execute()` не вернул результат возращает `$this->_response`

### How do controllers interact with another?

* [`_forward()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Action/Action.php#L127) и [`Magento\Framework\Controller\Result/Forward`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Controller/Result/Forward.php) — выполяет другой экшен в рамках текущего
* [`_redirect()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Action/Action.php#L157) и [`Magento\Framework\Controller\Result/Redirect`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Action/Action.php#L157) — редиректит на другой экшен

### How are different response types generated?

[Result object](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/routing.html#result-object)

Респонсы реализуют [`Magento\Framework\Controller\ResultInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Controller/ResultInterface.php)

В M2 есть такие типы:
* [`Controller\Result\Raw`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Controller/Result/Raw.php)
* [`Controller\Result\Json`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Controller/Result/Json.php)
* [`Controller\Result\Forward`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Controller/Result/Forward.php)
* [`Controller\Result\Redirect`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Controller/Result/Redirect.php)
* [`View\Result\Layout`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Result/Layout.php) — рендерит лаяут без `default` хендла и ляаута страницы (1-column etc.)
* [`View\Result\Page`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/View/Result/Page.php) — обарачивает лаяут в лаяут страницы
