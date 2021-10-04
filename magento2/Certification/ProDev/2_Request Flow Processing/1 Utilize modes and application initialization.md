# Utilize modes and application initialization

>Identify the steps for application initialization. How would you design a customization that should act on every request and capture output data regardless of the controller?

>Describe how to use Magento modes. What are pros and cons of using developer mode/production mode? When do you use default mode? How do you enable/disable maintenance mode?

>Describe front controller responsibilities. In which situations will the front controller be involved in execution, and how can it be used in the scope of customizations?

## Инициализация

1. [`app/bootstrap.php`](https://github.com/magento/magento2/blob/2.4/app/bootstrap.php)
  * автолоад композера
  * umask
  * измения таймзоны на UTC
  * [php precision](https://www.php.net/manual/ru/ini.core.php#ini.precision) 
2. [`\Magento\Framework\App\Bootstrap::create`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Bootstrap.php#L120)
  * подготовка автолоадов
  * подготовка _generation/Magento_
3. [`\Magento\Framework\App\Bootstrap::createApplication`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Bootstrap.php#L234)
  * вызывает `$this->objectManager->create()`
4. [`\Magento\Framework\App\Bootstrap::run`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Bootstrap.php#L255)
```php
\Magento\Framework\Profiler::start('magento');
$this->initErrorHandler();
$this->assertMaintenance();
$this->assertInstalled();
$response = $application->launch();
$response->sendResponse();
\Magento\Framework\Profiler::stop('magento');
```

## Describe how to use Magento modes.

### What are pros and cons of using developer mode/production mode?

[About Magento modes](https://devdocs.magento.com/guides/v2.4/config-guide/bootstrap/magento-modes.html)

Особености режимов:

* **default** — используется по умолчанию при развёртывании мадженты
  * Статичные файлы кешируются
  * Исключения не отображаются, они записываются в лог
  * Заголовки _X-Magento-*_ скрываются
* **developer** — используется для разработки на локальной машине разработчика
  * Кеширование статичных файлов отключено
  * Подробное логирование
  * Включана [автоматическая компиляция кода](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-compiler.html)
  * Расширенный дебаг
  * Показываются _X-Magento-*_ заголовки
  * Производительность более медленная
  * Ошибки отображаются
* **production** — используется в продакшен окружении
  * Исключения не отображаются, они записываются в лог
  * Статичные файлы берутся только из кеша
  * Предатвращает автоматическую кодо-компиляцию
  * Не позволяет включать/отключать типы кеша в админке, только через cli
* **maintenance** — используется для технических работ, например, при деплое
  * ридиректит все запросы на дефолтную страницу `Service Temporarily Unavailabl`
  * При включеном maintenance режиме, в директории _var/_ содержится файл-флаг `.maintenance.flag`
  * Можно настроить доступ для определённых IP.

### When do you use default mode?

Дефолтный режим используется по умолчанию при развёртывании мадженты. Он не оптимизирован для использования на продакшене или для использования нескольких серверов.

### How do you enable/disable maintenance mode?

* `maintenance:enable` и `maintenance:disable` — включает/отключает [maintenance mode](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-maint.html)

### Команды для работы с режимами:

* `deploy:mode:show` — отображает текущий режим
* `deploy:mode:set {mode} [-s|--skip-compilation]` — изменяет режим.

## FrontController

[Magento\Framework\App\FrontController](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/FrontController.php) 

* используется по умолючанию, задаётся в глобальном _app/etc/di.xml_
* управляет роутингом, список роутов предоставляются классом [`Magento\Framework\App\RouterList`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/RouterList.php) 

### Экшены контролера

В отличии от M1 в M2 каждый экшен прдеставляет из себя отдельный класс реализующий [`Magento\Framework\App\ActionInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/ActionInterface.php), который содержит один метод `execute()`. Также контролер должен реализоывать интерфесы которые определяют с какими HTTP-методами контроллер используется, например:
* [`Magento\Framework\App\Action\HttpGetActionInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Action/HttpGetActionInterface.php) ­— для GET
* [`Magento\Framework\App\Action\HttpPostActionInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Action/HttpPostActionInterface.php) — для POST
остальные можно посмотреть в [Magento/Framework/App/Action/](https://github.com/magento/magento2/tree/2.4/lib/internal/Magento/Framework/App/Action)

`execute()` должен возвращать [`Magento\Framework\Controller\ResultInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Controller/ResultInterface.php) или [`Magento\Framework\App\ResponseInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/ResponseInterface.php)

Результаты обрабатываются в [`Magento\Framework\App\Http::launch()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Http.php#L118-L125):
```php
if ($result instanceof ResultInterface) {
    $this->registry->register('use_page_cache_plugin', true, true);
    $result->renderResult($this->_response);
} elseif ($result instanceof HttpInterface) {
    $this->_response = $result;
} else {
    throw new \InvalidArgumentException('Invalid return type');
}
```

## Контролеры API

Для API используется другие контролеры:

* REST — [`Magento\Webapi\Controller\Rest`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Webapi/Controller/Rest.php)
* SOAP — [`Magento\Webapi\Controller\Soap`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Webapi/Controller/Soap.php)
