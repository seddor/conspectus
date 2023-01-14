# Demonstrate ability to use plugins

>Demonstrate how to design complex solutions using the plugin’s life cycle. How do multiple plugins interact, and how can their execution order be controlled? How do you debug a plugin if it doesn’t work?

>Identify strengths and weaknesses of plugins. What are the limitations of using plugins for customization? In which cases should plugins be avoided?

[Plugins (Interceptors)](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/plugins.html)

Плагины используются для изменения поведения публичных методов классов/интерфейсов, путём перехвата вызова функций и запуска кода до/после/вокруг вызова метода.

Плагины позволяют менять поведение методов, при этом не меняя сам класс, M2 вызывает плагины с соответствии с настроенным порядком сортировки.

## Ограничения

Плагины нельзя создать для:

* Final методов;
* Final классов;
* не публичных методов;
* Статчных методов;
* `__construct`;
* Виртуальных типов;
* Объектов инстанцированных раньше, чем `Magento\Framework\Interception`.

## Определение плагина

Плагины определяются в `di.xml`
```xml
<type name="{ObservedType}">
    <plugin name="{pluginName}" type="{PluginClassName}" sortOrder="1" disabled="false" />
</type>
```
Обязательно:
* `ObservedType` — класс или интерфейс для которого используется плагин
* `pluginName` — идентификатор плагина, так же используется для мерджа конфигов для плагина
* `PluginClassName` — имя класса или виртуального типа
Опционально:
* `sortOrder` — плагины для одного метода запускаются в этом порядке
* `disabled` — отключает плагин

Плагины расплогаются в директории _Plugin_ модуля.

Для того чтобы добавить плагин для метода `setName`, в классе плагина можно указать такие методы:

* `beforeSetName`
* `aroundSetName` 
* `afterSetName`

Если имя нужного метода начинается с `_` например `_construct()`, то 

* `before_construct`
* `around_construct`
* `after_construct`

### Before

Может использоваться для изменения передаваемых аргуметов в изменяемый метод, если аргументов несколько нужно вернуть массив, если аргументы не нужно изменять, то нужно вернуть `null`.
```php
public function beforeSetName(\Magento\Catalog\Model\Product $subject, $name)
{
    return ['(' . $name . ')'];
}
```

### After

Может использоваться для изменения результата вызова метода.
```php
public function afterGetName(\Magento\Catalog\Model\Product $subject, $result)
{
    return '|' . $result . '|';
}
```
After-метод имеет доступ к всем аргуметам расширяемого метода, M2 передаёт их в качестве параметров полсе резльтата, если метод ничего не возращает то в результате будет `null`.
```php
namespace My\Module\Plugin;

class AuthLogger
{
    private $logger;

    public function __construct(\Psr\Log\LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    /**
     * @param \Magento\Backend\Model\Auth $authModel
     * @param null $result
     * @param string $username
     * @return void
     * @SuppressWarnings(PHPMD.UnusedFormalParameter)
     */
    public function afterLogin(\Magento\Backend\Model\Auth $authModel, $result, $username)
    {
        $this->logger->debug('User ' . $username . ' signed in.');
    }
}
```
в after-методе не нужно определять все аргуметны расширяемого метода.

>Если аргумент расширяемого метода опционален, то в after-методе он тоже должен быть опциональным.

Все параметры начиня с 3го это параметры которые передаются оригинальному методу.

### Around

Запускается до и после расширяемого метода.

Around-методы советуют не использовать без особой надобности, т.к. их использование может сказаться на производительности.
```php
public function aroundSave(\Magento\Catalog\Model\Product $subject, callable $proceed)
{
    $someValue = $this->doSmthBeforeProductIsSaved();
    $returnValue = null;

    if ($this->canCallProceedCallable($someValue)) {
        $returnValue = $proceed();
    }

    if ($returnValue) {
        $this->postProductToFacebook();
    }

    return $returnValue;
    }
```
В around-методы также передаются параметры, как и в after, если в расщиряемый метод предаются параметры, то их нужно объявить и в around-методе, использую такие же типы и значения по умолчанию. 

Если аргументы не будт использоватся в around методе то можно передать их таким образом:
```php
public function aroundSave(\My\Module\Model\MyUtility $subject, callable $proceed, ...$args)
{
    //do something
    $proceed(...$args);
}
```

## Порядок вызова

Плагины вызываются в соответствии с `sortOrder`, когда для одного метода определено несколько методов плагина одного типа.

Порядок запуска:
* Before, запускается с наименьшим `sortOrder`:
 * Запускается каждый плагин с методом before
 * Запускается часть around до callable
* После выполянения запускается с наибольшим `sortOrder`:
 * Запусается вторая часть around, после callable
 * выполняются after методы

### Пример

||PluginA|PluginB|PluginC|Action|
|-|-|-|-|-|
|**sortOrder**|10|20|30||
|**before**|beforeDispatch()|beforeDispatch()|beforeDispatch()||
|**around(до callable)**||aroundDispatch() [first half]|aroundDispatch() [first half]||
|**оригнальный**||||dispatch()|
|**around(после callable)**||aroundDispatch() [second half]|aroundDispatch() [second half]||
|**after**|afterDispatch()|afterDispatch()|afterDispatch()||

Будет запущено в следующем порядке:
* PluginA::beforeDispatch()
* PluginB::beforeDispatch()
* PluginB::aroundDispatch() (Magento calls the first half until callable)
  * PluginC::beforeDispatch()
  * PluginC::aroundDispatch() (Magento calls the first half until callable)
    * Action::dispatch()
  * PluginC::aroundDispatch() (Magento calls the second half after callable)
  * PluginC::afterDispatch()
* PluginB::aroundDispatch() (Magento calls the second half after callable)
* PluginB::afterDispatch()
* PluginA::afterDispatch()

## Наслеодвание

Плагны для клссов имеющих наследников так-же будут вызываться и для них.
Например, если сделать плагин для `Magento\Catalog\Block\Product\AbstractProduct`, то методы плагина так-же будут вызваны для дочерних классов, таких как `Magento\Catalog\Block\Product\View`, `Magento\Catalog\Block\Product\ProductList\Upsell` и т.д.

Определения глобальных плагинов можно переопределять в скопах.

## Плагин на плагин

Можно сделать плагин на плагин, он настраивается так-же как обычный плагин, реализация при этом будет выглядеть пимерно так:
```php
//Оригинальный плагин
class Example2Plugin1
{
    public function beforeFormat(\Chapter1\Plugins\Model\MyModel $subject, $name)
    {
        return [$name . ', Calia (original plugin)'];
    }
}
//Плагин на плагин
class Example2Plugin2
{
    public function beforeBeforeFormat(
        \Chapter1\Plugins\Model\Plugins\Example2Plugin1 $subject,
        \Chapter1\Plugins\Model\MyModel $model,
        $name
    ) {
        return [$model, $name . ', Elissa (plugin modifier)'];
    }
}
```

## Подробнее

[Alan Storm - Magento 2 Object Manager Plugin System](https://alanstorm.com/magento_2_object_manager_plugin_system/)
[Magent 2 observer or Plugin?](https://www.yireo.com/blog/1875-magent-2-observer-or-plugin)
