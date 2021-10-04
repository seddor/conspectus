# Configure event observers and scheduled jobs

>Demonstrate how to configure observers. How do you make your observer only be active on the frontend or backend?

>Demonstrate how to configure a scheduled job. Which parameters are used in configuration, and how can configuration interact with server configuration?

>Identify the function and proper use of automatically available events, for example *_load_after, etc.

## События и обработчики

[Events and observers](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/events-and-observers.html)

Обработка событий в M2 схожа с тем, что было в M1. Для работы с событиями (events) обработчиками (observers) в M2 используется патерн издатель-подписчик или [наблюдатель](https://refactoring.guru/ru/design-patterns/observer).

### События

События вызываются от модулей когда происходит определённое действие, которое его тригерит, также можно создавать собственные ивенты.

События может быть вызвано с помощью [`Magento\Framework\Event\Manager`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Event/Manager.php), объект класса может быть инстансирован с помощью DI (через конструктор класса).

Для вызова события вызвается метод `dispath` в который передаётся идентификатор события и может быть передан массив с любыми объектами или данными.
```php
namespace MyCompany\MyModule;

use Magento\Framework\Event\ObserverInterface;

class MyClass
{
  /**
   * @var EventManager
   */
  private $eventManager;

  public function __construct(\Magento\Framework\Event\Manager $eventManager)
  {
    $this->eventManager = $eventManager;
  }

  public function something()
  {
    $eventData = null;
    // Code...
    $this->eventManager->dispatch('my_module_event_before');
    // More code that sets $eventData...
    $this->eventManager->dispatch('my_module_event_after', ['myEventData' => $eventData]);
  }
}
```

Обработчики для события указываются в файлах `events.xml`, события могут обрабатываться глобально или в определённых area:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Event/etc/events.xsd">
    <event name="my_module_event_before">
        <observer name="myObserverName" instance="MyCompany\MyModule\Observer\MyObserver" />
    </event>
    <event name="my_module_event_after">
        <observer name="myObserverName" instance="MyCompany\MyModule\Observer\AnotherObserver" />
    </event>
</config>

```
* `name` (обязательно) - идентификатор обрабатываемого события
* `instance` (обязательно) - Класс обработчика
* `disabled` - позволяет отключить обработчик, можно использовать если нужно отключить какой-либо обработчик, нужно создать зависимый модуль и в нём указать имя обработчика и `"disabled"=true`
* `shared` - Определляет жизненый цикл обработчика, по умолчанию `true`, т.е. обработчик создаётся как синглтон.

### Обработчики

Обработчик вызывается в [`Magento\Framework\Event\Invoker\InvokerDefault`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Event/Invoker/InvokerDefault.php)

В M2, в отличии от M1 для каждой подписки на событие должен использоваться свой класс обработчика. Обработчики должны лежать в _`<module-dir>`/Observers_ и реализовывать [`Magento\Framework\Event\ObserverInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Event/ObserverInterface.php), в интерфейсе определён метод `execute`, в котором должна происходить обработка события.
```php
namespace MyCompany\MyModule\Observer;

use Magento\Framework\Event\ObserverInterface;

class AnotherObserver implements ObserverInterface
{
  public function __construct()
  {
    // Observer initialization code...
    // You can use dependency injection to get any class this observer may need.
  }

  public function execute(\Magento\Framework\Event\Observer $observer)
  {
    $myEventData = $observer->getData('myEventData');
    // Additional observer execution code...
  }
}
```

### Советы по использованию

[Observers Best Practices](https://devdocs.magento.com/guides/v2.4/ext-best-practices/extension-coding/observers-bp.html)

* Делать обработчики эффективными. Нужно стараться делать обработчики насколько можно лёгковесными и избегать в них сложных вычислений, т.к. это может сказаться на скорости работы приложения.
* Не включать бизнес логику
* В обработчике должна быть логика только для его работы, остальная логика должна выноситься в другие классы, которые можно использовать в обработчике с помощью DI.
* Объявлять обработчики в нужных облостях видимости.
* Избегать цикличного вызова событий.
* Обработчики не должны полагаться на порядок их вызова.

Так-же из [гайдланов мадженты](https://devdocs.magento.com/guides/v2.4/coding-standards/technical-guidelines.html)

* Все значения и объекты, передаваемые в ивенты, **не должны** изменяться в обработчиках, для измения ввода или вывода методов нужно использовать плагины(1.5).

### Дополнительно

[Список всех вызваемых событий в M2](https://cyrillschumacher.com/magento-2.3-list-of-all-dispatched-events/)

## Scheduled jobs

M2 позволяет запускать джобы по крону.

### Настройка джобы

[Custom cron job and cron group reference](https://devdocs.magento.com/guides/v2.4/config-guide/cron/custom-cron-ref.html)
[Configure a custom cron job and cron group (tutorial)](https://devdocs.magento.com/guides/v2.4/config-guide/cron/custom-cron-tut.html)

#### Группы

В M2 для джоб появились группы, группы нужны для разделения выполнения джоб. Для большинства джоб можно использовать группу `default`, для индексов используется группа `index`.

Конфигурируются джобы в `crontab.xml`:
```xml
<config>
    <group id="<group_name>">
        <job name="<job_name>" instance="<classpath>" method="<method>">
            <schedule>[time]</schedule>
            <config_path>[path]</config_path>
        </job>
    </group>
</config>
```
* group_name — идентфикатор группы
* job_name — уникальный идентификатор джобы
* classpath — имя класса обработчика джобы, классы хранятся в _`<module-dir>`/Cron_
* method — метод обработчика
* schedule — время запуска в формате [cron выражения](https://crontab.guru/). Не указывается, если выражение берёться из базы или другого источника.
* config_path — путь до конфига содержащего cron-выражение

Ноды config_path и schedule взаимозаменяемые, в конфиге может присутовать любяах из них.

##### Кастомные группы

Можно создавать кастомные группы, они настраиваются в `cron_groups.xml`:
```xml
<config>
    <group id="<group_name>">
        <schedule_generate_every>1</schedule_generate_every>
        <schedule_ahead_for>4</schedule_ahead_for>
        <schedule_lifetime>2</schedule_lifetime>
        <history_cleanup_every>10</history_cleanup_every>
        <history_success_lifetime>60</history_success_lifetime>
        <history_failure_lifetime>600</history_failure_lifetime>
        <use_separate_process>1</use_separate_process>
    </group>
</config>
```
всё время указывается в минутах
* `schedule_generate_every` — Частота с которой джобы пишутся в таблицу `cron_schedule`.
* `schedule_ahead_for` — время до начала работы за котрое джобы пишутся в `cron_schedule`.
* `schedule_lifetime` — время в течении которого джоба может быть запущена, после него она будет пропущена, если не успела запуститься.
* `history_cleanup_every` — время в течении которого джоба хранится в истории.
* `history_success_lifetime` — время в течении которого, послн удачного завершённая, джоба хранится в истории.
* `history_failure_lifetime` — время в течении которого джоба, неудачно завершённая, хранится в истории.
* `use_separate_process` — запуск джоб группы в отдельных процессах

### Запуск 

Запуск выполяентся командой `magento cron:run [–group=”"]`

Порядок вызова:

* [`Magento\Cron\Console\Command::execute`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Cron/Console/Command/CronCommand.php#L91)
* [`Magento\Framework\App\Cron::launch`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Cron.php#L76)
* Вызов события `default`
* [`Magento\Cron\Observer\ProcessCronQueueObserver`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Cron/Observer/ProcessCronQueueObserver.php)
* Проверка джоб для определённых групп
* Если при запуске был указан параметр `--bootstrap=standaloneProcessStarted=1`, то группа запускается отдельно
* Для групп вызываются
  * `$this->cleanupJobs($groupId, $currentTime)`
  * `$this->generateSchedules($groupId)`
  * `$this->processPendingJobs($groupId, $jobsRoot, $currentTime)`
