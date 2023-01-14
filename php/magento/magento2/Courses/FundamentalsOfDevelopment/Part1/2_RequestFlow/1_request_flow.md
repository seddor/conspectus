# Флоу запроса

Флоу запроса — последовательность шагов выполняемых приложением для обработки и ответа посылаемых к нему запросов.

## Высокоуровневый роутинг

Стандратный флоу запроса в M2 выглядит так (1-4 выполняются для каждого запроса):

1. index.php — является точкой входа, куда попадают запросы от веб-сервера. В index.php происохдит инциализация `\Magento\Framework\App\Bootstrap`.
2. Bootstrap — настраивает некоторые параметры PHP и окружения, инциализирует `\Magento\Framework\AppInterface`, в случае веба это `\Magento\Framework\App\Http`, запускает процесс выполнения приложения
3. App — грузит конфигурацию и инициирует front контроллер
4. Routing — запускается процесс роутинга, в цикле поверяется каждый роут пока не будет найден тот, который соответствует запросу
5. Controller processing — запрос попадает в найденый для него класс-контроллер, в котором происходит обработка запроса
6. Rendering — рендеринг генерирует нужные блоки и подготавливает их вывод
7. Flushing output — то что было отрендорено возвщрается в ответе

# Роутинг

Routers (маршрутизаторы) — классы ответственные за распознавание различных типов роутов, на основе URL. Роутеры преобразуют URL запроса в тот, который маджента может обработать и находят классы которые будут их обрабатывать.

## Основые цели

* Определение всех доступных роутов
* Преобразование URL в URL обрабатываемые маджетой
* Парсинг параметров запроса
* Индентифицироване классов-контроллеров, которые будут обрабатывать URL

## Класс-контроллер

В M2, классы-контролеры отличаются от M1, здесь каждый класс контроллер обрабатывает только 1 конкретный экшен. Классы-контроллеры реализуют интерфейс [`Magento\Framework\App\ActionInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/ActionInterface.php), он содержит один метод `execute()`.

## Роутер

Роутер — класс предназначеный для обработки определённого типа URL. Обработка заключается в поиске нужного класса-контроллера.

### Magento-style URL

В M2 URL состоят из 3х частей, всё остальное является параметрами, как и в M1. Пример: _catalog/product/view/id/5_

Для обработки других видов URL, можно создать кастомный роутер.

## Определения роута

Роуты определяются в `routers.xml`.

# Контроллеры

Контроллеры являются часть концепции MVC, они отвечают за:

* Обработка запроса и его параметров
* Начало процесса рендеринга (View).
* При необходимости инстанцирование моделей (Model).

## Примеры

Взаимодействие с запросом:
```php
public function execute()
{
    //Get init data from request
    $categoryId = (int) $this->getRequest()->getParam('category', false);
    $productId = (int) $this->getRequest()->getParam('id');
    $specifyOptions = $this->getRequest()->getParam('options');
    if ($this->getRequest()->isPost() && $this->getRequest()->getParam(self::PARAM_NANE_URL_ENCODED)) {
        $thos->_initProduct()
        //some code
    }
    //some code
}
```
Взаимодействие с моделямиы:
```php
protected function _initProduct()
{
    //Get init data from request
    $categoryId = (int) $this->getRequest()->getParam('category', false);
    $productId = (int) $this->getRequest()->getParam('id');
    
    $params = new \Magento\Freamwork\DataObject();
    $params->setCateoryId($categoryId);
    
    $product = $this->_objectManager->get(\Magento\Catalog\Helper\Product::class);
    return $product->initProduct($productId, $this, $params)
}
```
Рендеринг
```php
try {
    $page = $this->resultPageFactory->created(false, ['isIsolared' => true]);
    $this->viewHelper->prapareAndRender($page, $productId, $this, $params);
    return $page;
} catch (Excecption $e) {
    if ($e->getCode() == $this->viewHelper->ERR_NO_PRODUCT_LOADED) {
        return $this->noProductRedirect();
    }
    $this->_objectManager->get('Psr\Log\LoggerInterface')->critical($e);
    $resultForward = $this->resultForwardFactory->created();
    $resultForward->forward('noroute');
    return $resultForward;
}
```

# Рендеринг и вывод

## Флоу

1. Контроллер инициализирует процесс рендеринга
2. Система рендеринга генерирует список блоков на основе лаяута
3. Система рендеринга генерирует вывод блоков

В общих чертах в процессе рендеринга происходит следующие, после чего происходит вывод результата:

1. Front controller [`Magento\Framework\App\FrontController`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/FrontController.php) — обрабатывает запрос и получает объект результата.
2. App [`Magento\Framework\App\Http`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Http.php) — в методе `launch()` копирует вывод в объект ответа.
3. Bootstrap [`Magento\Framework\App\Bootstrap`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Bootstrap.php) — отправляет вывод из объекта ответа клиенту.

Процесс рендеринга и вывода ([`Magento\Framework\App\Bootstrap::run()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Bootstrap.php#L253)):
```php
\Magento\Framework\Profiler::start('magento');
$this->initErrorHandler();
$this->assertMaintenance();
$this->assertInstalled();
$response = $application->launch();
$response->sendResponse();
\Magento\Framework\Profiler::stop('magento');
```
[`Magento\Framework\App\Http::launch()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Http.php#L109)
```php
public function launch()
    {
        $areaCode = $this->_areaList->getCodeByFrontName($this->_request->getFrontName());
        $this->_state->setAreaCode($areaCode);
        $this->_objectManager->configure($this->_configLoader->load($areaCode));
        /** @var \Magento\Framework\App\FrontControllerInterface $frontController */
        $frontController = $this->_objectManager->get(\Magento\Framework\App\FrontControllerInterface::class);
        $result = $frontController->dispatch($this->_request);
        // TODO: Temporary solution until all controllers return ResultInterface (MAGETWO-28359)
        if ($result instanceof ResultInterface) {
            $this->registry->register('use_page_cache_plugin', true, true);
            $result->renderResult($this->_response);
        } elseif ($result instanceof HttpInterface) {
            $this->_response = $result;
        } else {
            throw new \InvalidArgumentException('Invalid return type');
        }
        if ($this->_request->isHead() && $this->_response->getHttpResponseCode() == 200) {
            $this->handleHeadRequest();
        }
        // This event gives possibility to launch something before sending output (allow cookie setting)
        $eventParams = ['request' => $this->_request, 'response' => $this->_response];
        $this->_eventManager->dispatch('controller_front_send_response_before', $eventParams);
        return $this->_response;
    }
```
Контроллер после выполнения возвращет объект [`Magento\Framework\Controller\ResultInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Controller/ResultInterface.php). 

>Сейчас контроллеры могут возвращать и другие ответы (не ResultInterface), но эта возможность будут удалена в будущем, сейчас это оставленно для совместимости.
