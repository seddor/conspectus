# Архитектура контроллера

Контроллер — это класс предназначеный для рабты с урлом или группой урлов. В M2 контроллер может обрабатывать только один экшен.

## Структура класса

Классы-контроллеры реализуют интерфейс [`Magento\Framework\App\ActionInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/ActionInterface.php), он содержит один метод `execute()`.

Обычно класс-контролера имеет такую иерархию (от текущего класса контролера):

1. extends [`Magento\Framework\App\Action\Action`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Action/Action.php)
2. extends [`Magento\Framework\App\Action\AbstractAction`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Action/AbstractAction.php)
3. implements [`Magento\Framework\App\ActionInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/ActionInterface.php)

`AbstractAction` класс:
* получают объекты для запроса и ответа (в конструкторе, через DI).
* содержат методы `getRequest()` и `getResponse()`

`Action` класс:
* реализует метод `dispatch()` (который вызывает роутер), в котором вызывается метод `execute` и выбрасываются стандартные ивенты для флоу запроса:
    * `controller_action_predispatch`
    * `'controller_action_predispatch_' . $request->getRouteName()`
    * `'controller_action_predispatch_' . $request->getFullActionName()`
    * `'controller_action_postdispatch_' . $request->getFullActionName()`
    * `'controller_action_postdispatch_' . $request->getRouteName()`
    * `controller_action_postdispatch`

## Объект ответа

Объект ответа реализуют [`Magento\Framework\Controller\ResultInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Controller/ResultInterface.php)

Основные классы для объектов ответа:

* [`View\Result\Page`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/View/Result/Page.php) — ренддерит HTML использую механизм лаяутов.
```php
public function __construct(\Magento\Framework\View\Result\PageFactory $pageFactory)
{
    $this->pageFactory = $pageFactory;
}

public function execute()
{
    $page = $this->pageFactory->create();
    return $page;
}
```
* [`Controller\Result\Json`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Controller/Result/Json.php) — возвращает JSON.
```php
public function __construct(\Magento\Framework\Controller\Result\JsonFactory $jsonFactory)
{
    $this->jsonFactory = $jsonFactory;
}

public function execute()
{
    $result = $this->jsonFactory->create();
    $data = ['foo' => 'bar'];
    return $result->setData($data);
}
```
* [`Controller\Result\Forward`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Controller/Result/Forward.php) — используется для перенапрвления выполнения обработки другим контроллером, без редиректа.
```php
public function __construct(\Magento\Framework\Controller\Result\ForwardFactory $forwardFactory)
{
    $this->forwardFactory = $forwardFactory;
}

public function execute()
{
    $result = $this->forwardFactory->create();
    $result->forward('some/new/route');
    return $result;
}
```
* [`Controller\Result\Redirect`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Controller/Result/Redirect.php) — используется для возврата редиректа.
```php
public function __construct(\Magento\Framework\Controller\Result\RedirectFactory $redirectFactory)
{
    $this->redirectFactory = $redirectFactory;
}

public function execute()
{
    $result = $this->redirectFactory->create();
    $result->forward('some/new/url');
    return $result;
}
```

### forward

метод [`_forward()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Controller/Result/Forward.php#L77):
```php
protected function _forward($action, $controller = null, $module = null, array $params = null)
{
        $request = $this->getRequest();
        $request->initForward();
        if (isset($params)) {
            $request->setParams($params);
        }
        if (isset($controller)) {
            $request->setControllerName($controller);

            // Module should only be reset if controller has been specified
            if (isset($module)) {
                $request->setModuleName($module);
            }
        }
        $request->setActionName($action);
        $request->setDispatched(false);
```
метод работает с объектом запоса: меняет ему даннные контролера на нужные

### refirect

метод [`_redirect()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Controller/Result/Forward.php#L77):
```php
protected function _redirect($path, $arguments = [])
{
    $this->_redirect->redirect($this->getResponse(), $path, $arguments);
    return $this->getResponse();
}
```
метод работает с ответом.

# Admin & Frontend контроллеры

Основное отлчие между frontend и backend (admin) контроллерами, то что backend-контроллеры должны поддерживать защищённый доступ (ACL).

Backend-контроллеры имеют схожую с front-end иерархию, только в неё добавляется ещё 2 класса:

1. extends [`Magento\Backend\App\AbstractAction`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Backend/App/AbstractAction.php)
2. extends [`Magento\Backend\App\Action`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Backend/App/Action.php)

`AbstractAction` класс:
* расщиряет конструктор, добавляя некоторые объекты
* расширяет методы: `dispatch()`, `_forward()`, `_redirect()`
* содержит метод `_isAllowed()` и другие необходимые для работы в админке

[конструктор `AbstractAction`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Backend/App/AbstractAction.php#L90):
```php
public function __construct(Action\Context $context)
{
    parent::__construct($context);
    $this->_authorization = $context->getAuthorization();
    $this->_auth = $context->getAuth();
    $this->_helper = $context->getHelper();
    $this->_backendUrl = $context->getBackendUrl();
    $this->_formKeyValidator = $context->getFormKeyValidator();
    $this->_localeResolver = $context->getLocaleResolver();
    $this->_canUseBaseUrl = $context->getCanUseBaseUrl();
    $this->_session = $context->getSession();
}
```
